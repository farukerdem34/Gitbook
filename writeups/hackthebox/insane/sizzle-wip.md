# Sizzle (WIP)

## Gaining Access&#x20;

Nmap scan:

```
$ nmap -p- --min-rate 5000 10.129.73.156   
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-26 00:20 EDT
Nmap scan report for 10.129.73.156
Host is up (0.0075s latency).
Not shown: 65507 filtered tcp ports (no-response)
PORT      STATE SERVICE
21/tcp    open  ftp
53/tcp    open  domain
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
443/tcp   open  https
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
5985/tcp  open  wsman
5986/tcp  open  wsmans
9389/tcp  open  adws
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49668/tcp open  unknown
49671/tcp open  unknown
49688/tcp open  unknown
49689/tcp open  unknown
49692/tcp open  unknown
49695/tcp open  unknown
49700/tcp open  unknown
49713/tcp open  unknown
```

Another AD machine.&#x20;

### Anonymous FTP

This machine allows for anomyous FTP access, but there's nothing present in the server:

```
$ ftp 10.129.73.156
Connected to 10.129.73.156.
220 Microsoft FTP Service
Name (10.129.73.156:kali): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password: 
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
229 Entering Extended Passive Mode (|||56291|)
125 Data connection already open; Transfer starting.
226 Transfer complete.
```

Let's move on for now, we might need this later.&#x20;

### HTTP

This just shows a GIF of bacon sizzling:

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

When viewing the page source, this is located at the `/images` directory, which we don't have access to. However, it does tell us this is an IIS server based on the error page:

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

### SMB Shares

`enum4linux` works with `guest` credentials.

```bash
$ enum4linux -u 'guest' -p '' -a 10.129.73.156
<TRUNCATED>
        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        CertEnroll      Disk      Active Directory Certificate Services share
        Department Shares Disk      
        IPC$            IPC       Remote IPC
        NETLOGON        Disk      Logon server share 
        Operations      Disk      
        SYSVOL          Disk      Logon server share 

```

`smbmap` can confirm this:

```
$ smbmap -u 'guest' -p '' -H 10.129.73.156                          
[+] IP: 10.129.73.156:445       Name: 10.129.73.156                                     
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        CertEnroll                                              NO ACCESS       Active Directory Certificate Services share
        Department Shares                                       READ ONLY
        IPC$                                                    READ ONLY       Remote IPC
        NETLOGON                                                NO ACCESS       Logon server share 
        Operations                                              NO ACCESS
        SYSVOL                                                  NO ACCESS       Logon server share
```

Within the Department Shares, there are loads of directories:

```bash
$ smbclient -U 'guest' '//10.129.73.156/Department Shares'  
Password for [WORKGROUP\guest]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue Jul  3 11:22:32 2018
  ..                                  D        0  Tue Jul  3 11:22:32 2018
  Accounting                          D        0  Mon Jul  2 15:21:43 2018
  Audit                               D        0  Mon Jul  2 15:14:28 2018
  Banking                             D        0  Tue Jul  3 11:22:39 2018
  CEO_protected                       D        0  Mon Jul  2 15:15:01 2018
  Devops                              D        0  Mon Jul  2 15:19:33 2018
  Finance                             D        0  Mon Jul  2 15:11:57 2018
  HR                                  D        0  Mon Jul  2 15:16:11 2018
  Infosec                             D        0  Mon Jul  2 15:14:24 2018
  Infrastructure                      D        0  Mon Jul  2 15:13:59 2018
  IT                                  D        0  Mon Jul  2 15:12:04 2018
  Legal                               D        0  Mon Jul  2 15:12:09 2018
  M&A                                 D        0  Mon Jul  2 15:15:25 2018
  Marketing                           D        0  Mon Jul  2 15:14:43 2018
  R&D                                 D        0  Mon Jul  2 15:11:47 2018
  Sales                               D        0  Mon Jul  2 15:14:37 2018
  Security                            D        0  Mon Jul  2 15:21:47 2018
  Tax                                 D        0  Mon Jul  2 15:16:54 2018
  Users                               D        0  Tue Jul 10 17:39:32 2018
  ZZ_ARCHIVE                          D        0  Mon Jul  2 15:32:58 2018
```

The `Users` directory gives me a list of usernames to work with and perhaps ASREP-Roast:

```
  amanda                              D        0  Mon Jul  2 15:18:43 2018
  amanda_adm                          D        0  Mon Jul  2 15:19:06 2018
  bill                                D        0  Mon Jul  2 15:18:28 2018
  bob                                 D        0  Mon Jul  2 15:18:31 2018
  chris                               D        0  Mon Jul  2 15:19:14 2018
  henry                               D        0  Mon Jul  2 15:18:39 2018
  joe                                 D        0  Mon Jul  2 15:18:34 2018
  jose                                D        0  Mon Jul  2 15:18:53 2018
  lkys37en                            D        0  Tue Jul 10 17:39:04 2018
  morgan                              D        0  Mon Jul  2 15:18:48 2018
  mrb3n                               D        0  Mon Jul  2 15:19:20 2018
  Public                              D        0  Wed Sep 26 01:45:32 2018
```

The `ZZ_Archive` directory contains a lot of different files which all have the same size and edit date with different extensions.&#x20;

```
smb: \ZZ_ARCHIVE\> ls
  .                                   D        0  Mon Jul  2 15:32:58 2018
  ..                                  D        0  Mon Jul  2 15:32:58 2018
  AddComplete.pptx                    A   419430  Mon Jul  2 15:32:58 2018
  AddMerge.ram                        A   419430  Mon Jul  2 15:32:57 2018
  ConfirmUnprotect.doc                A   419430  Mon Jul  2 15:32:57 2018
  ConvertFromInvoke.mov               A   419430  Mon Jul  2 15:32:57 2018
  ConvertJoin.docx                    A   419430  Mon Jul  2 15:32:57 2018
  CopyPublish.ogg                     A   419430  Mon Jul  2 15:32:57 2018
  <TRUNCATED>
```

All of these folders were rather useless and contained nothing of interest.

WIP!&#x20;
