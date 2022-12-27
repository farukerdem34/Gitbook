# Knife

## Gaining Access

Nmap scan:

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

### PHP-Dev

I found nothing interesting about the web application hosted on port 80. However, when viewing the traffic proxied through Burpsuite, we see an interesting header:

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

PHP/8.1.0-dev is vulnerable to a publicly available RCE exploit.

{% embed url="https://www.exploit-db.com/exploits/49933" %}

This script can be used to gain a shell as the user.

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

From this, we can get a shell using the `mkfifo` script. After spawning a TTY shell, we find that we are the user `james`.

## Privilege Escalation

### Knife Sudo

When enumerating sudo privileges, we see that we can run `knife`.

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

`knife` can be used to spawn a root shell since we can run `sudo` with it.

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>
