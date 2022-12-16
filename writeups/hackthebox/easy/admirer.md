# Admirer

## Gaining Access

Nmap scan:

<figure><img src="../../../.gitbook/assets/image (86).png" alt=""><figcaption></figcaption></figure>

An Nmap vuln scan reveals there is a `robots.txt` file on port 80.

<figure><img src="../../../.gitbook/assets/image (82).png" alt=""><figcaption></figcaption></figure>

We can head to that directly.

### FTP Credentials

The robots.txt file gave us the `/admin-dir` directory.

<figure><img src="../../../.gitbook/assets/image (94).png" alt=""><figcaption></figcaption></figure>

However, visiting it gives us a 403.

<figure><img src="../../../.gitbook/assets/image (119).png" alt=""><figcaption></figcaption></figure>

I used `gobuster` on this directory and found a `credentials.txt` file.

<figure><img src="../../../.gitbook/assets/image (77).png" alt=""><figcaption></figcaption></figure>

We can view this file to find some FTP credentials.

<figure><img src="../../../.gitbook/assets/image (78).png" alt=""><figcaption></figcaption></figure>

Then, we can login to FTP.

### HTML Dump File

Within FTP, we can find a backup of the website.

<figure><img src="../../../.gitbook/assets/image (76).png" alt=""><figcaption></figcaption></figure>
