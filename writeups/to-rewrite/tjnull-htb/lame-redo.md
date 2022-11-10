# Lame (Redo)

Lame

Tuesday, 1 February 2022

1:16 am

&#x20;

**\[Lame HTB]{.underline}**

10.10.10.3

FTP

SSH&#x20;

SMB shares open

FTP and SSH require a password.

SMB does not allow for null sessions.

21/tcp   open  ftp         vsftpd 2.3.4

22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)

139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)

445/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)

3632/tcp open  distccd?

There is a distccd port open... which means nothing

After that we can use this exploit to expose the weakness and gain an initial shell.

[https://github.com/amriunix/CVE-2007-2447/blob/master/usermap\_script.py](https://github.com/amriunix/CVE-2007-2447/blob/master/usermap\_script.py)
