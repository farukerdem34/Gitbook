# Enumeration (2)

Enumeration

Friday, July 15, 2022

8:17 AM

Nmap scan:

!\[\[121\_Enumeration\_image001.png]]

&#x20;

Nmap detailed scan:

{width="8.729166666666666in" height="0.6979166666666666in"}

&#x20;

!\[\[121\_Enumeration\_image003.png]]

&#x20;

!\[\[121\_Enumeration\_image004.png]]

There is an interesting MSSQL\_BAK.rar file.

&#x20;

FTP Anonymous Login:

{width="7.614583333333333in" height="2.2916666666666665in"}

&#x20;

Downloading the MSSQL\_BAK:

!\[\[121\_Enumeration\_image006.png]]

&#x20;

Opening the rar file:

!\[\[121\_Enumeration\_image007.png]]

&#x20;

We need a password for this file.

&#x20;

Converting to hash and cracking using rar2john and john:

!\[\[121\_Enumeration\_image008.png]]

&#x20;

Opening rar file:

!\[\[121\_Enumeration\_image009.png]]

&#x20;

&#x20;

&#x20;
