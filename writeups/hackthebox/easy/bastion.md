# Bastion

## Gaining Access

Nmap scan revealed a lot of ports:

<figure><img src="../../../.gitbook/assets/image (61).png" alt=""><figcaption></figcaption></figure>

### Windows Backup

When checking the SMB shares, we fnd that the Backups share does not require credentials to access.

<figure><img src="../../../.gitbook/assets/image (10) (2) (2).png" alt=""><figcaption></figcaption></figure>

When checking the files, we can see that there's a WindowsImageBackup directory within it.

<figure><img src="../../../.gitbook/assets/image (13) (2) (3).png" alt=""><figcaption></figcaption></figure>

Within it, there was a few .vhd files.

<figure><img src="../../../.gitbook/assets/image (62).png" alt=""><figcaption></figcaption></figure>

These file can actually be mounted and they do not requrie credentials at all!

<figure><img src="../../../.gitbook/assets/image (8) (2) (2).png" alt=""><figcaption></figcaption></figure>

From here, because this is a Windows backup, we can directly go to the `C:\Windows\System32\config` file to use the SYSTEM and SAM registry folders and dump the credentials via `samdump2`.

<figure><img src="../../../.gitbook/assets/image (72).png" alt=""><figcaption></figcaption></figure>

Then, this hash is easily cracked using hashcat.

<figure><img src="../../../.gitbook/assets/image (10) (2).png" alt=""><figcaption></figcaption></figure>

Afterwards, we can SSH in as the L4mpje user.

## Privilege Escalation

### mRemoteNG

In the user's directory, I enumerated all the directories that I could using `dir /all`.

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (4).png" alt=""><figcaption></figcaption></figure>

Within the `AppData\Roaming` folder, we can find some files related to mRemoteNG.

<figure><img src="../../../.gitbook/assets/image (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

Checking the confCons.xml file, we can find an encrypted password for the Administrator.

<figure><img src="../../../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption></figcaption></figure>

mRemoteNG passwords can be decrypted using this repository:

{% embed url="https://github.com/haseebT/mRemoteNG-Decrypt" %}

<figure><img src="../../../.gitbook/assets/image (2) (1) (6).png" alt=""><figcaption></figcaption></figure>

Then, we can SSH in as the administrator using this password.
