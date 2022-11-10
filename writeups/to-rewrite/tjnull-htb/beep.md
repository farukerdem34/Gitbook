# Beep

Beep

Tuesday, 1 February 2022

1:17 am

**\[Beep]{.underline}**

Regular Nmap.

Nmap scan report for 10.10.10.7

Host is up (0.012s latency).

Not shown: 988 closed ports

PORT      STATE SERVICE

22/tcp    open  ssh

25/tcp    open  smtp

80/tcp    open  http

110/tcp   open  pop3

111/tcp   open  rpcbind

143/tcp   open  imap

443/tcp   open  https

993/tcp   open  imaps

995/tcp   open  pop3s

3306/tcp  open  mysql

4445/tcp  open  upnotifyp

10000/tcp open  snet-sensor-mgmt

Web server first, there is a login page and stuff we have to access somehow.&#x20;

22/tcp    open  ssh        OpenSSH 4.3 (protocol 2.0)

25/tcp    open  smtp       Postfix smtpd

80/tcp    open  http       Apache httpd 2.2.3

110/tcp   open  pop3       Cyrus pop3d 2.3.7-Invoca-RPM-2.3.7-7.el5\_6.4

111/tcp   open  rpcbind    2 (RPC #100000)

143/tcp   open  imap       Cyrus imapd 2.3.7-Invoca-RPM-2.3.7-7.el5\_6.4

443/tcp   open  ssl/https?

878/tcp   open  status     1 (RPC #100024)

993/tcp   open  ssl/imap   Cyrus imapd

995/tcp   open  pop3       Cyrus pop3d

3306/tcp  open  mysql      MySQL (unauthorized)

4190/tcp  open  sieve      Cyrus timsieved 2.3.7-Invoca-RPM-2.3.7-7.el5\_6.4 (included w/cyrus imap)

4445/tcp  open  upnotifyp?

4559/tcp  open  hylafax    HylaFAX 4.3.10

5038/tcp  open  asterisk   Asterisk Call Manager 1.1

10000/tcp open  http       MiniServ 1.570 (Webmin httpd)

There are two login pages, one on 80 and the other is on 10000.

Webmin 1.570 is running on it, and there's a possibility of brute forcing the damn thing.&#x20;

Unfortunately, I got blocked for trying any of those passwords listed by Hydra.

There is a need to reset the machine in order to get back to where I was.&#x20;

Searching up Elastix would reveal there is an LFI exploit. When I try to exploit it, I can see that there is a FreePBX file being displayed.&#x20;

There are a few passwords within the thing.&#x20;

**jEhdIekWmdjE**

**Passw0rd**

**Amp109**

**Amp111**

Root plus the first password worked! I tried SSH too and it worked out just fine!

There are other methods which we can use, it seems.

There are other ways to RCE via elastix via **18650.py**.

From there we are able to get the flag from it, and we can execute a ton of shit to easily escalate.&#x20;

Either that or we can execute commands based on the root based admin access.

We can also use shellshock through varying the User Agent header.

These are all good ways of which we can exploit it. The key is identifying that root is the password into the machine.
