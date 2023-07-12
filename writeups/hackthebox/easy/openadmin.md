# OpenAdmin

## Gaining Access

Nmap scan:

<figure><img src="../../../.gitbook/assets/image (133).png" alt=""><figcaption></figcaption></figure>

### Port 80

We can run a `gobuster` scan on port 80:

<figure><img src="../../../.gitbook/assets/image (110) (1) (3).png" alt=""><figcaption></figcaption></figure>

I visited the `/music` directory first and it brought me to some corporate website:

<figure><img src="../../../.gitbook/assets/image (130) (1).png" alt=""><figcaption></figcaption></figure>

When I clicked `login`, it brought me to `/ona`, which was a dashboard for OpenNetAdmin:

<figure><img src="../../../.gitbook/assets/image (96) (4).png" alt=""><figcaption></figcaption></figure>

This version of OpenNetAdmin was vulnerable to RCE:

<figure><img src="../../../.gitbook/assets/image (124) (1).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://www.exploit-db.com/exploits/47691" %}

We can gain a shell by following the PoC:

<figure><img src="../../../.gitbook/assets/image (88) (4).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (105) (3).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

### Jimmy Credentials

<figure><img src="../../../.gitbook/assets/image (129) (2).png" alt=""><figcaption></figcaption></figure>

The users present on the machine are `joanna` and `jimmy`, and it seems that `ssh` with this password works on `jimmy`:

<figure><img src="../../../.gitbook/assets/image (106) (4).png" alt=""><figcaption></figcaption></figure>

### Joanna SSH Key

When looking around the `/var/www/internal` directory, we can some code referencing the private SSH key of `joanna`:

<figure><img src="../../../.gitbook/assets/image (132) (2).png" alt=""><figcaption></figcaption></figure>

When reading further, we can find the password and username hard-coded into the application:

<figure><img src="../../../.gitbook/assets/image (128) (2).png" alt=""><figcaption></figcaption></figure>

The hash can be cracked to give `Revealed`. We can read `/etc/apache2/sites-available/internal.conf` to find the hidden sub-domain and port it is open on:

<figure><img src="../../../.gitbook/assets/image (118) (3).png" alt=""><figcaption></figcaption></figure>

After port forwarding via `ssh -L 52846:127.0.0.1:52846 jimmy@10.10.10.71`, we can access the login page:

<figure><img src="../../../.gitbook/assets/image (90) (1).png" alt=""><figcaption></figcaption></figure>

Logging in reveals a password protected RSA Private Key:

<figure><img src="../../../.gitbook/assets/image (99) (3).png" alt=""><figcaption></figcaption></figure>

We can decrypt this via `ssh2john`:

<figure><img src="../../../.gitbook/assets/image (103) (3).png" alt=""><figcaption></figcaption></figure>

Afterwards, we can use `openssl rsa -in key -out privatekey` to write the private RSA key, then `ssh` in as `joanna`:

<figure><img src="../../../.gitbook/assets/image (114) (3).png" alt=""><figcaption></figcaption></figure>

### Nano GTFOBins

When checking `sudo` privileges, `joanna` can run `nano` as `root`:

<figure><img src="../../../.gitbook/assets/image (112) (3).png" alt=""><figcaption></figcaption></figure>

Based on GTFOBins, we can simply run the following commands to gain a shell:

<figure><img src="../../../.gitbook/assets/image (94) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (134).png" alt=""><figcaption></figcaption></figure>

Rooted!
