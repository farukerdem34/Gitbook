# Tally

Tally

Sunday, June 19, 2022

7:39 PM

Nmap scan:

!\[\[23\_Tally\_image001.png]]

&#x20;

!\[\[23\_Tally\_image002.png]]

&#x20;

Port 80:

!\[\[23\_Tally\_image003.png]]

&#x20;

This is Microsoft Shared Point.

We can sign in if we had credentials.

!\[\[23\_Tally\_image004.png]]

&#x20;

Within the page source, we can identify a potential version that is being used here.

!\[\[23\_Tally\_image005.png]]

&#x20;

Based on the time this box was released and also the version, I would guess this is something like Microsoft Sharepoint 2015.

For now, we will keep this in mind as we enumerate more.

&#x20;

Gobuster scan:

!\[\[23\_Tally\_image006.png]]

We can do a check on the different pages that are present with the version included.

This did not help much as it just spewed a bunch of different directories we could go into.

&#x20;

Eventually, after googling, I found this one directory that contained a file.

!\[\[23\_Tally\_image007.png]]

&#x20;

This contained a document called FTP-details.

!\[\[23\_Tally\_image008.png]]

&#x20;

When viewed and downloaded, this is what shows.

!\[\[23\_Tally\_image009.png]]

&#x20;

When we view the site pages, which has one item, we find another document.

&#x20;

!\[\[23\_Tally\_image0010.png]]

&#x20;

!\[\[23\_Tally\_image0011.png]]

&#x20;

There are some usernames here. When trying them with the password, it appears that ftp\_user works.

{width="5.8125in" height="3.90625in"}

&#x20;

FTP:

Within the /user/tim/files directory, we can find this interesting little bit:

!\[\[23\_Tally\_image0013.png]]

&#x20;

There's a keepass credentials storage here. We can try to brute force this.

!\[\[23\_Tally\_image0014.png]]

&#x20;

There's also this other file that tells us what this file does.

{width="3.1354166666666665in" height="2.0in"}

&#x20;

{width="7.75in" height="2.6666666666666665in"}

&#x20;

{width="9.197916666666666in" height="3.0416666666666665in"}

&#x20;

Enumerating KeePass:

{width="5.989583333333333in" height="1.9270833333333333in"}

&#x20;

Within this, we can find one password.

!\[\[23\_Tally\_image0019.png]]

&#x20;

Check Credentials:

{width="7.958333333333333in" height="1.6770833333333333in"}

&#x20;

Now we have access to this share.

Within this share, there are some documents and passwords for SQL that are considered old.

!\[\[23\_Tally\_image0021.png]]

&#x20;

Within one of the directories, we can find a load of binaries:

!\[\[23\_Tally\_image0022.png]]

&#x20;

&#x20;

!\[\[23\_Tally\_image0023.png]]

&#x20;

When we download that and bring it back to our machine, we can see this little thing here.

!\[\[23\_Tally\_image0024.png]]

&#x20;

There are some credentials within this!

With this, we can proceed with the RCE below.

&#x20;

Port 1433:

After finding credentials, we can login with MSSQL using sqsh. We can then gain a shell using xp\_cmdshell.

&#x20;

{width="6.0625in" height="0.6979166666666666in"}

&#x20;

!\[\[23\_Tally\_image0026.png]]

&#x20;

!\[\[23\_Tally\_image0027.png]]

&#x20;

So we have RCE through the MSSQL server. Now we can gain a reverse shell.

&#x20;

Shell:

!\[\[23\_Tally\_image0028.png]]

&#x20;

!\[\[23\_Tally\_image0029.png]]

&#x20;

!\[\[23\_Tally\_image0030.png]]

&#x20;

{width="5.375in" height="2.1666666666666665in"}

&#x20;

Flag:

!\[\[23\_Tally\_image0032.png]]

6e61436de1e808356188d28cc1038a16

&#x20;

&#x20;

PE:

!\[\[23\_Tally\_image0033.png]]

&#x20;

SeImpersonatePrivilege.

&#x20;

We can just use PrintSpoofer.

{width="7.75in" height="2.5in"}

&#x20;

Flag:

!\[\[23\_Tally\_image0035.png]]

b17e77f9569a8697691cebea5b4ab6f4

&#x20;

Pretty weird box with little weird enumerations that needed to be done.

&#x20;
