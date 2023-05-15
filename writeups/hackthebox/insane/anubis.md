# Anubis

## Gaining Access

Nmap scan:

```
$ nmap -p- --min-rate 5000 -Pn 10.129.190.189
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-15 04:21 EDT
Nmap scan report for 10.129.95.208
Host is up (0.024s latency).
Not shown: 65531 filtered tcp ports (no-response)
PORT      STATE SERVICE
135/tcp   open  msrpc
443/tcp   open  https
445/tcp   open  microsoft-ds
593/tcp   open  http-rpc-epmap
49703/tcp open  unknown
```

### Windcorp --> SSTI

`crackmapexec` can be used to enumerate domain name.&#x20;

{% code overflow="wrap" %}
```
$ crackmapexec smb 10.129.190.189                                       
[*] completed: 100.00% (1/1)
SMB         10.129.190.189  445    EARTH            [*] Windows 10.0 Build 17763 x64 (name:EARTH) (domain:windcorp.htb) (signing:True) (SMBv1:False)
```
{% endcode %}

Visiting `windcorp.htb` does not show anything though. In this case, we can do a sub-domain enumeration and find something.&#x20;

```
$ wfuzz -c -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -H 'Host:FUZZ.windcorp.htb' --hc=404 -u https://windcorp.htb  
Total requests: 100000

=====================================================================
ID           Response   Lines    Word       Chars       Payload                     
=====================================================================

000000001:   200        1007 L   3245 W     46774 Ch    "www"
```

Add the domains to the `/etc/hosts` file, and `www` is visited it shows a typical corporate page:

<figure><img src="../../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

When we fill in the Contact and try to submit, it let's us preview our submission at `preview.asp`.&#x20;

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Here's the HTTP request for the `Send` function, which is a GET request to `save.asp`:

```http
GET /save.asp?name=test&email=test%40website.com&subject=test&message=test HTTP/2
Host: www.windcorp.htb
Cookie: ASPSESSIONIDAEDSDSSQ=PAFDFIPAFOLMPNMELMKFNMFL
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: https://www.windcorp.htb/
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Te: trailers

```

Interesting. Since this is using `asp`, and the output is printed on another screen, we can test some SSTI. I used this payload:

<figure><img src="../../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

And this was returned:

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

This confirms that SSTI is possible on the machine.&#x20;
