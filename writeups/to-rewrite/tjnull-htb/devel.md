---
description: Straightforward and really simple machine.
---

# Devel

## Gaining Access

Nmap scan:

<figure><img src="../../../.gitbook/assets/image (129) (1) (2).png" alt=""><figcaption></figcaption></figure>

When checking the FTP server, we can determine that anonymous credentials work.

### Webroot FTP

Through an Nmap scan with the `--script vuln` flag, we can view the directory automatically.

<figure><img src="../../../.gitbook/assets/image (7) (1) (7).png" alt=""><figcaption></figcaption></figure>

Based on the files that are present here, we can see that port 80 is running a IIS server, and this directory has an `aspnet_client` folder within it. This FTP server could potentially provide us with  read and write access to the webroot directory.

So, I created a quick `aspx` reverse shell using `msfvenom`.

<figure><img src="../../../.gitbook/assets/image (33) (2) (2).png" alt=""><figcaption></figcaption></figure>

Then, we can put this file in the FTP directory.

<figure><img src="../../../.gitbook/assets/image (22) (1) (3) (1).png" alt=""><figcaption></figcaption></figure>

Afterwards, we can simply run the reverse shell via `curl`.

<figure><img src="../../../.gitbook/assets/image (36) (1) (4).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

### JuicyPotato

When in the machine, we can first check our privileges.

<figure><img src="../../../.gitbook/assets/image (11) (2) (4).png" alt=""><figcaption></figcaption></figure>

Seeing that we have the SeImpersonatePrivilege privilege enabled, we can use JuicyPotato.exe to exploit this easily with another `.exe` reverse shell generated using `msfvenom`.&#x20;

<figure><img src="../../../.gitbook/assets/image (33) (6).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (29) (2) (2).png" alt=""><figcaption></figcaption></figure>
