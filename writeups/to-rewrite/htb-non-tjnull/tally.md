# Tally

## Gaining Access

Nmap scan results:

<figure><img src="../../../.gitbook/assets/image (34) (4).png" alt=""><figcaption></figcaption></figure>

The most interesting was port 1433 with MSSQL and port 21 with FTP, both of which should not be exposed.

### Sharepoint

Checking port 80 reveals a Microsoft Sharepoint instance.

<figure><img src="../../../.gitbook/assets/image (47) (1) (2).png" alt=""><figcaption></figcaption></figure>

For Microsoft Sharepoint, we can visit the `viewlsts.aspx` file to see all the site contents.

<figure><img src="../../../.gitbook/assets/image (40) (4).png" alt=""><figcaption></figcaption></figure>

The Documents had one folder which contained FTP credentials:

<figure><img src="../../../.gitbook/assets/image (43) (4) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (4) (7).png" alt=""><figcaption></figcaption></figure>

The Site Pages had another folder which contained the FTP username:

<figure><img src="../../../.gitbook/assets/image (57) (3) (1).png" alt=""><figcaption></figcaption></figure>

With these, we can access the FTP server.

<figure><img src="../../../.gitbook/assets/image (50) (1) (2).png" alt=""><figcaption></figcaption></figure>

### FTP Keepass

Within the FTP server, I found a KeePass database within the `/user/tim/files` directory.

<figure><img src="../../../.gitbook/assets/image (39) (5).png" alt=""><figcaption></figcaption></figure>

We can download this KeePass database back, and then use `keepass2john` to convert it to a hash and crack it with `john`.

<figure><img src="../../../.gitbook/assets/image (21) (1) (4).png" alt=""><figcaption></figcaption></figure>

Using this password, we can access the database via `kpcli`.&#x20;

<figure><img src="../../../.gitbook/assets/image (15) (1) (2) (2).png" alt=""><figcaption></figcaption></figure>

Then, we can use `show -f 0` to view all the passwords available.

<figure><img src="../../../.gitbook/assets/image (26) (1).png" alt=""><figcaption></figcaption></figure>

I tested these credentials with `smbmap`, and we can read one share:

<figure><img src="../../../.gitbook/assets/image (10) (5) (1).png" alt=""><figcaption></figcaption></figure>

### SMB Shares + RCE

Within the SMB shares, we can find some documents containing MSSQL passwords that do not work however. There are also a ton of bnaries witin the `zz_Migration\Binaries` folder.

Looking around it, wecome across a suspicious `tester.exe` that does not fit in with the rest.

<figure><img src="../../../.gitbook/assets/image (59) (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

We can download this back to our machine for analysing, and I used `strings` as I always do on binaries I get. When looking through the output, we can see a few lines that contains MSSQL credentials:

<figure><img src="../../../.gitbook/assets/image (52) (1) (1).png" alt=""><figcaption></figcaption></figure>

With this, we can access the MSSQL instance with `sqsh`. Afterwards, getting RCE via `xp_cmdshell` was trivial.

<figure><img src="../../../.gitbook/assets/image (55) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (7) (3) (3).png" alt=""><figcaption></figcaption></figure>

Then, we can gain a reverse shell as this user with whatever method.

<figure><img src="../../../.gitbook/assets/image (66) (1).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

### PrintSpoofer

When enumerating the privileges this user has, we notice that they have the SeImpersonatePrivilege token enabled.

<figure><img src="../../../.gitbook/assets/image (63) (1).png" alt=""><figcaption></figcaption></figure>

We can easily use `printspoofer.exe` to spawn an administrator shell.

<figure><img src="../../../.gitbook/assets/image (12) (5) (1).png" alt=""><figcaption></figcaption></figure>
