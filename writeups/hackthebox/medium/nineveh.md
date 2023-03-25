# Nineveh

## Gaining Access

Nmap scan:

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

### Port 80

I ran a `gobuster` scan against the web application on port 80 and found a hidden directory:

<figure><img src="../../../.gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>

When we visit the `/department` directory, we are presented with a login page:

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

There was really nothing I could do with this page, so I moved on.

### Port 443

I ran another `gobuster` scan on this website, and found some directories:

```
$ gobuster dir -k -u https://10.10.10.43 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -t 150
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            https://10.10.10.43
[+] Method:         GET
[+] Threads:        150
[+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt
[+] Status codes:   404
[+] User Agent:     gobuster/3.1.0
[+] Timeout:        10s
===============================================================
2022/02/05 22:00:49 Starting gobuster in directory enumeration mode
===============================================================
/db (Status: 301)
/server-status (Status: 403)
/secure_notes (Status: 301)
```

There was a hidden `/db` directory. When we visit it, we see a phpLiteAdmin instance:

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

This was another dead end however since we don't have credentials.

### Brute Force

Normally, I don't brute force passwords unless it's my last lead, and at this juncture I had no leads. Brute forcing the logins on both pages were successful with username `admin`.&#x20;

<figure><img src="../../../.gitbook/assets/image (43).png" alt=""><figcaption><p><em>HTTP Login Brute Force</em></p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (12).png" alt=""><figcaption><p><em>HTTPS Login Brute Force</em></p></figcaption></figure>

With this, we can login to both of the services. Port 80 revealed an image:

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

We can instantly tell that this is vulnerable to LFI because of the `notes` parameter in the URL. Next, we can view the phpLiteAdmin admin dashboard:

<figure><img src="../../../.gitbook/assets/image (44).png" alt=""><figcaption></figcaption></figure>

This version of phpLiteAdmin is vulnerable to RCE, and we can follow the PoC here:

{% embed url="https://www.exploit-db.com/exploits/24044" %}

However, this exploit requires us to be able to run the PHP file somehow, and the LFI on the port 80 dashboard allows for that since it is also in PHP.

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

Now that we have RCE, getting a shell is trivial.&#x20;

## Privilege Escalation

### Chrootkit

When we move to the main directory, we can find a `/report` directory present, and within it confirmation that the machine has a rootkit installed.

<figure><img src="../../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

This root kit has some local privilege escalation exploits.&#x20;

{% embed url="https://vk9-sec.com/chkrootkit-0-49-local-privilege-escalation-cve-2014-0476/" %}

We can use this to gain a `root` shell.

<figure><img src="../../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

![](<../../../.gitbook/assets/image (36).png>)
