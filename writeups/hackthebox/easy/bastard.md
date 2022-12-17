# Bastard

## Gaining Access

Nmap scan:

<figure><img src="../../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

Running a scan on port 80 reveals that this is a Drupal 7 instance running.

<figure><img src="../../../.gitbook/assets/image (53).png" alt=""><figcaption></figcaption></figure>

### Drupalgeddon

Checking the CHANGELOG.txt file, we find that this is running Drupal 7.54.

<figure><img src="../../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

With this, we can run the Drupalgeddon2 exploit.

{% embed url="https://www.exploit-db.com/exploits/44449" %}

When run, this would spawn a shell for us and we can gain a reverse shell via the nc64.exe method.

## Privilege Escalation

### MS15-051 Enum

When checking the `systeminfo` output, we see that this is running Windows Server 2008 R2 Datacenter, which was vulnerable to the MS15-051 exploit.

<figure><img src="../../../.gitbook/assets/image (60).png" alt=""><figcaption></figcaption></figure>

Here's the working exploit:

{% embed url="https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS15-051" %}

We can download this to the machine to execute commands to give us an administrator shell.

<figure><img src="../../../.gitbook/assets/image (54).png" alt=""><figcaption></figcaption></figure>
