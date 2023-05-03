---
description: My second box!
---

# Lame

## Gaining Access

Nmap scan:

```
$ nmap -p- --min-rate 5000 -Pn 10.129.205.55
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-03 11:24 EDT
Nmap scan report for 10.129.205.55
Host is up (0.0079s latency).
Not shown: 65530 filtered tcp ports (no-response)
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3632/tcp open  distccd
```

```
$ sudo nmap -p 21,22,139,445,3632 -sC -sV -O -T4 10.129.205.55
[sudo] password for kali: 
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-03 11:26 EDT
Nmap scan report for 10.129.205.55
Host is up (0.0076s latency).

PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.13
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
22/tcp   open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 600fcfe1c05f6a74d69024fac4d56ccd (DSA)
|_  2048 5656240f211ddea72bae61b1243de8f3 (RSA)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
```

### SMB RCE --> Root Shell

The SMB version is outdated and vulnerable to loads of exploits. We can easily use `msfconsole` to exploit this.

<figure><img src="../../../.gitbook/assets/image (474).png" alt=""><figcaption></figcaption></figure>
