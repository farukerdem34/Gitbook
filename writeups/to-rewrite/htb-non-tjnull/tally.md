# Tally

## Gaining Access

Nmap scan results:

<figure><img src="../../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

The most interesting was port 1433 with MSSQL and port 21 with FTP, both of which should not be exposed.

### Sharepoint

Checking port 80 reveals a Microsoft Sharepoint instance.

<figure><img src="../../../.gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>

For Microsoft Sharepoint, we can visit the `viewlsts.aspx` file to see all the site contents.

<figure><img src="../../../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

The Documents had one folder which contained FTP credentials:

<figure><img src="../../../.gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

The Site Pages had another folder which contained the FTP username:

<figure><img src="../../../.gitbook/assets/image (57).png" alt=""><figcaption></figcaption></figure>

With these, we can access the FTP server.

<figure><img src="../../../.gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure>

### FTP Keepass

Within the FTP server, I found a KeePass database within the `/user/tim/files` directory.

<figure><img src="../../../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

We can download this KeePass database back, and then use `keepass2john` to convert it to a hash and crack it with `john`.

<figure><img src="../../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

Using this password, we can access the database via `kpcli`.&#x20;

<figure><img src="../../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

Then, we can use `show -f 0` to view all the passwords available.

<figure><img src="../../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

I tested these credentials with `smbmap`, and we can read one share:

<figure><img src="../../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

### SMB Shares + RCE

Within the SMB shares, we can find some documents containing MSSQL passwords that do not work however. There are also a ton of bnaries witin the `zz_Migration\Binaries` folder.

Looking around it, wecome across a suspicious `tester.exe` that does not fit in with the rest.

<figure><img src="../../../.gitbook/assets/image (59).png" alt=""><figcaption></figcaption></figure>

We can download this back to our machine for analysing, and I used `strings` as I always do on binaries I get. When looking through the output, we can see a few lines that contains MSSQL credentials:

<figure><img src="../../../.gitbook/assets/image (52).png" alt=""><figcaption></figcaption></figure>

With this, we can access the MSSQL instance with `sqsh`. Afterwards, getting RCE via `xp_cmdshell` was trivial.

<figure><img src="../../../.gitbook/assets/image (55).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

Then, we can gain a reverse shell as this user with whatever method.

<figure><img src="../../../.gitbook/assets/image (66).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

### PrintSpoofer

When enumerating the privileges this user has, we notice that they have the SeImpersonatePrivilege token enabled.

<figure><img src="../../../.gitbook/assets/image (63).png" alt=""><figcaption></figcaption></figure>

We can easily use `printspoofer.exe` to spawn an administrator shell.

<figure><img src="../../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>
