# Slort

Slort

Wednesday, 16 March 2022

3:31 pm

The machine IP is 192.168.61.53.

My IP is 192.168.49.61.

&#x20;

!\[\[88\_Slort\_image001.png]]

Interesting.

&#x20;

!\[\[88\_Slort\_image002.png]]

&#x20;

FTP did not allow anonymous logins.

Enum4linux returned nothing,

&#x20;

Port 8080 returned this:

!\[\[88\_Slort\_image003.png]]

&#x20;

Well, I ran another directory enumeration on this thing.

!\[\[88\_Slort\_image004.png]]

PHP!

&#x20;

Looking at exploits for XAMPP, there were quite a few:

!\[\[88\_Slort\_image005.png]]

&#x20;

I went to the site.

&#x20;

!\[\[88\_Slort\_image006.png]]

Now I was wondering if the page parameter could be used for some directory traversal or SQL injection.

&#x20;

!\[\[88\_Slort\_image007.png]]

&#x20;

There was actually an LFI, whereby we can include URLs within that.

!\[\[88\_Slort\_image008.png]]

&#x20;

From there, we can just download some form of Powershell Script.

&#x20;

!\[\[88\_Slort\_image009.png]]

Powershell does not work, so we have to try other methods.

&#x20;

We can include PHP shells, and this can be done easily.

!\[\[88\_Slort\_image0010.png]]

&#x20;

!\[\[88\_Slort\_image0011.png]]

Let's try some multiple step exploits.

Create a quick shell.exe using MSFvenom.

{width="10.729166666666666in" height="1.7395833333333333in"}

Now, we just need to download this onto the web server.

&#x20;

!\[\[88\_Slort\_image0013.png]]

&#x20;

This php shell would do that, as we can execute code through php using this LFI.

&#x20;

!\[\[88\_Slort\_image0014.png]]

&#x20;

Afterwards, simply execute this file.

&#x20;

!\[\[88\_Slort\_image0015.png]]

&#x20;

!\[\[88\_Slort\_image0016.png]]

&#x20;

Grab the local.txt

!\[\[88\_Slort\_image0017.png]]

&#x20;

Within the main directory, there's this Backup file.

!\[\[88\_Slort\_image0018.png]]

&#x20;

Within this, there seems to be some information regarding some stuff.

!\[\[88\_Slort\_image0019.png]]

I transferred all of these files to my own machine using smb servers.

&#x20;

Backup.txt contained a lot of stuff which I did not understand at first.

!\[\[88\_Slort\_image0020.png]]

&#x20;

Info.txt contained descriptions of a cronjob:

!\[\[88\_Slort\_image0021.png]]

&#x20;

Seems this is a TFTP server trying to get something out of another server. I wondered if I could change this cronjob somehow.

Ran a winpeas and confirmed this:

{width="10.96875in" height="0.5729166666666666in"}

This clearly has something to do with stuff.

&#x20;

As such, we can simply replace this TFTP.EXE with our own malicious stuff!

&#x20;

Generate another exe file called TFTP.EXE and port it to the machine.

!\[\[88\_Slort\_image0023.png]]

&#x20;

{width="3.9375in" height="0.6354166666666666in"}

&#x20;

Afterwards, rename the old TFTP.EXE to something else.

&#x20;

!\[\[88\_Slort\_image0025.png]]

Then put our shell there and just wait. The cronjob should execute sooner or later.

!\[\[88\_Slort\_image0026.png]]

&#x20;

!\[\[88\_Slort\_image0027.png]]

&#x20;
