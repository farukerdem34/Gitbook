---
description: Good beginners machine.
---

# Cap

## Gaining Access

We begin with an Nmap scan:

<figure><img src="../../../.gitbook/assets/image (207).png" alt=""><figcaption></figcaption></figure>

Did a detailed Nmap scan to find outmore about the services:

<figure><img src="../../../.gitbook/assets/image (218).png" alt=""><figcaption></figcaption></figure>

The FTP server did not allow for any anonymous logins, so we can head to the HTTP servers.

### Port 80

The webpage just revealed a dashboard for some security stats.

<figure><img src="../../../.gitbook/assets/image (186).png" alt=""><figcaption></figcaption></figure>

Trying to install the **second PCAP** file in the **Security Snapshot** directory directs us to the `/2` endpoint. We can change this to the `/0` endpoint to download a different file.

### PCAP Analysis

PCAP analysis here reveals that there is some FTP traffic without encryption that reveals some credentials.

<figure><img src="../../../.gitbook/assets/image (222).png" alt=""><figcaption></figcaption></figure>

With this, we can SSH as `nathan` into the machine.

## Privilege Escalation

### Cap\_setuid

I ran LinPEAS to find some Escalation Vectors. Python has the `cap_setuid` bit set, which allows us to call the `os.setuid()` function. We can use this to spawn a root shell accordingly.

<figure><img src="../../../.gitbook/assets/image (2) (1) (6) (2).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (184).png" alt=""><figcaption></figcaption></figure>
