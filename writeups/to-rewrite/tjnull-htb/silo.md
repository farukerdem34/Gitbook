# Silo

Silo

Monday, 7 February 2022

11:31 pm

This is a Windows Machine.

My IP is 10.10.16.5.

The target IP is 10.10.10.82.

&#x20;

Preliminary nmap scan shows HTTP, MSRPC, DS and Oracle.

Ran nikto, gobuster and a more detailed nmap scan as well.

&#x20;

!\[\[22\_Silo\_image001.png]]

&#x20;

Windows Server 2008 R2, could be useful to take note of for priv escalation using a certain potato.

The entire scan for directories was pointless, because everything returned a 400 status, which is basically useless.

!\[\[22\_Silo\_image002.png]]

SMB is also kinda useless, as we cannot list anything.

&#x20;

The last thing to check is the Oracle database, which is something I've never seen before. I ran a more detailed nmap scan on it

&#x20;

!\[\[22\_Silo\_image003.png]]

&#x20;

Okay, so I know Oracle is a database. And it seems that unauthorized tag could be useful of sorts. I found this github on hacktricks., called odat.git. This is an Oracle penetration testing tool. Sounds like what I need.

&#x20;

Within the files there is a odat.py.

&#x20;

I ran it, just to check what's up.

Took a while, but this was the correct thing to use. Seems to enumerate lots of things for us to use. There's an XE and XEXDB valid service name, which his basically used to connect to the database. So we need to use this name in order to connect to an instance of the database.

{width="10.375in" height="6.0625in"}

&#x20;

Next, it seems to have found valid credentials in scott:tiger. I suppose it tests the default usernames and passwords to check which works.

!\[\[22\_Silo\_image005.png]]

&#x20;

So we have an SID and a username and password, let's see what we can do with this. There is an ability for RCE using either Java, Schedular or External Tables.

&#x20;

Seems that Java does not work.

{width="10.40625in" height="1.3125in"}

&#x20;

Dbmschedular works! ( I coped and pasted some shit, no idea what I just did lul)

{width="14.322916666666666in" height="1.65625in"}

&#x20;

Ok I have no idea what I just did I just know it works.

&#x20;

[https://0xdf.gitlab.io/img/ODAT\_main\_features\_v2.0.jpg](https://0xdf.gitlab.io/img/ODAT\_main\_features\_v2.0.jpg)

This is a useful image to relate to the different things that we are able to do. There isn't a way to run commands, but we can upload files, and that sounds like an msfvenom kind of job.

&#x20;

Created a quick shell using the exe file, because IIS is something that tends to run on aspx. Idk, I tried both exe and aspx. Reading the documentation of the tool, there is a **utlfile and dbmsxslprocessor** module that can be used to upload files to the webserver.

&#x20;

Let's try it. We get an insufficient privileges error.

{width="12.666666666666666in" height="1.2708333333333333in"}

&#x20;

Interesting. Read online, looked at walkthroughs and they all highlighted the fact that we would need to have the **--sysdba flag.** This is because of the fact that the user scott is not given any privileges, and including this flag would basically allow us to do more funny things!

&#x20;

This worked, and I managed to upload the file directly into the web directory.

&#x20;

{width="12.4375in" height="1.1458333333333333in"}

&#x20;

From here, it seems possible that we can execute this file from the odat.py tool.

&#x20;

{width="12.75in" height="1.09375in"}

For some reason I get a connection and this does not seem to work. Let's try other methods, although I'm really tired.

!\[\[22\_Silo\_image0011.png]]

All other methods I tried to use did not work, could not get a connection.

&#x20;

To solve the box I just used the same thing to read the flags...boo...

&#x20;

&#x20;
