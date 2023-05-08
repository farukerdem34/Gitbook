# Undetected (WIP)

## Gaining Access

Nmap scan:

```
$ nmap -p- --min-rate 5000 10.129.136.44
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-08 12:00 EDT
Nmap scan report for 10.129.136.44
Host is up (0.010s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

### DJewelry --> PHPUnit RCE

The website is a company page:

<figure><img src="../../../.gitbook/assets/image (500).png" alt=""><figcaption></figcaption></figure>

If we try to Visit Store, we get redirected to `store.djewelry.htb`. The store page is identical to the original, but it has more functionalities:

<figure><img src="../../../.gitbook/assets/image (621).png" alt=""><figcaption></figcaption></figure>

I tried adding products to the cart and maybe finding an exploit pertaining to that, but it was disabled.

<figure><img src="../../../.gitbook/assets/image (630).png" alt=""><figcaption></figcaption></figure>

Since there's no functionalities on this site, let's run a `feroxbuster` directory scan.

```
$ gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://store.djewelry.htb -x php -t 100 
===============================================================
Gobuster v3.3
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://store.djewelry.htb
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.3
[+] Extensions:              php
[+] Timeout:                 10s
===============================================================
2023/05/08 12:18:37 Starting gobuster in directory enumeration mode
===============================================================
/.php                 (Status: 403) [Size: 283]
/index.php            (Status: 200) [Size: 6215]
/images               (Status: 301) [Size: 325] [--> http://store.djewelry.htb/images/]
/login.php            (Status: 200) [Size: 4129]
/products.php         (Status: 200) [Size: 7447]
/cart.php             (Status: 200) [Size: 4396]
/css                  (Status: 301) [Size: 322] [--> http://store.djewelry.htb/css/]
/js                   (Status: 301) [Size: 321] [--> http://store.djewelry.htb/js/]
/vendor               (Status: 301) [Size: 325] [--> http://store.djewelry.htb/vendor/]
/fonts                (Status: 301) [Size: 324] [--> http://store.djewelry.htb/fonts/]
```

When we view the `/vendor` endpoint, we see a file system with different PHP libraries:

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Searching for exploits for each of them leads me an RCE for PHPUnit:

{% embed url="https://www.exploit-db.com/exploits/50702" %}

This works:

<figure><img src="../../../.gitbook/assets/image (434).png" alt=""><figcaption></figcaption></figure>

Then, use a `bash` one-liner to get a reverse shell.

<figure><img src="../../../.gitbook/assets/image (639).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

### Steven Shell

We can't access the user flag yet, and `steven` is the user of this machine. Within the `/var/backups` folder, there's a file that is not meant to be there:

```
www-data@production:/var/backups$ ls -la
total 72
drwxr-xr-x  2 root     root      4096 May  8 15:07 .
drwxr-xr-x 13 root     root      4096 Feb  8  2022 ..
-rw-r--r--  1 root     root     34011 Feb  8  2022 apt.extended_states.0
-r-x------  1 www-data www-data 27296 May 14  2021 info
```

This file was an ELF binary:

{% code overflow="wrap" %}
```
www-data@production:/var/backups$ file info
info: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=0dc004db7476356e9ed477835e583c68f1d2493a, for GNU/Linux 3.2.0, not stripped
```
{% endcode %}

We can transfer this back to my machine for some reverse engineering via `ghidra`. There were loads of functions within the binary.

WIP.
