# Craft (2)

Craft2

Saturday, July 9, 2022

1:11 AM

Nmap scan:

!\[\[113\_Craft2\_image001.png]]

&#x20;

&#x20;

&#x20;

&#x20;

Port 80:

!\[\[113\_Craft2\_image002.png]]

&#x20;

File upload, as well as accepting a certain file format only.

!\[\[113\_Craft2\_image003.png]]

&#x20;

ODT files basically means we have to use LibreOffice to generate some form of malicious shell.

I got lazy to think of ways to make a file, so I used MSF.

&#x20;

However, something interesting was shown when we generated and uploaded some form of payload.

!\[\[113\_Craft2\_image004.png]]

&#x20;

This tells me that there is no way of which to use Macro attempts again, because they probably disabled macros on their devices.

&#x20;

We can opt to use ODT files to instead reference files from our file system through SMB, and be able to steal the hash through that method.

&#x20;

!\[\[113\_Craft2\_image005.png]]

&#x20;

WE can use this one exploit.

{width="6.21875in" height="4.770833333333333in"}

&#x20;

Once uploaded, we would receive this on the responder.

{width="9.885416666666666in" height="2.71875in"}

&#x20;

We now have credentials.

{width="9.760416666666666in" height="2.71875in"}

&#x20;

{width="9.822916666666666in" height="1.3333333333333333in"}

&#x20;

SMB:

{width="9.822916666666666in" height="2.1979166666666665in"}

&#x20;

WebApp:

{width="8.260416666666666in" height="2.9791666666666665in"}

&#x20;

Uploading command shell:

!\[\[113\_Craft2\_image0012.png]]

&#x20;

!\[\[113\_Craft2\_image0013.png]]

&#x20;

Shell:

!\[\[113\_Craft2\_image0014.png]]

&#x20;

!\[\[113\_Craft2\_image0015.png]]

&#x20;

{width="6.96875in" height="2.2604166666666665in"}

&#x20;

Flag:

!\[\[113\_Craft2\_image0017.png]]

&#x20;

There are other users on this machine, aka thecybergeek which we probably need to access somehow.

There also could be the chance that we have to re-use the credentials we found earlier.

&#x20;

winPEAS:

!\[\[113\_Craft2\_image0018.png]]

&#x20;

I also ran PowerUp, because I thought we have to escalate to the other user.

&#x20;

!\[\[113\_Craft2\_image0019.png]]

&#x20;

So this user is running that process.

However, we cannot restart the machine from this shell, hence this cannot be used.

&#x20;

When looking at the path, we can see this file:

{width="9.854166666666666in" height="3.1979166666666665in"}

&#x20;

This is the first file that is used, and I found it odd because why would it have a junction link within here? It points to the same place.

Anyways, we can use RunasCs to gain another shell as thecybergeek with the earlier password.

&#x20;

!\[\[113\_Craft2\_image0021.png]]

&#x20;

This does nothing for us though.

So let's move onto other things regarding this machine.

&#x20;

Checking on ports:

!\[\[113\_Craft2\_image0022.png]]

&#x20;

There is a port 3306 that is open on the machine, and we can enumerate the MySQL instance that is running on it.

We can use chisel to forward it out.

&#x20;

Chisel Forwarding:

!\[\[113\_Craft2\_image0023.png]]

&#x20;

Then let's look for some credentials.

&#x20;

{width="8.479166666666666in" height="2.53125in"}

&#x20;

WE can login as root, meaning the process is likely being run by the administrator user.

!\[\[113\_Craft2\_image0025.png]]

&#x20;

This means we can load UDF and stuff perhaps.

!\[\[113\_Craft2\_image0026.png]]

&#x20;

!\[\[113\_Craft2\_image0027.png]]

&#x20;

The load\_file function isn't present on the server, and as such we cannot run the UDF functions.

We can't even load the plugins because well, there isn't a plugins folder.

!\[\[113\_Craft2\_image0028.png]]

&#x20;

Which is a bit weird.

We cannot change the plugin directory as well, leaving us with little stuff here.

We do have a phpmyadmin database, but it's also empty.

&#x20;

Since we can load and write files, what we can do is use WerTrigger.

&#x20;

!\[\[113\_Craft2\_image0029.png]]

&#x20;

Then using the mysql instance, we can copy the file to C:\windows\system32.

&#x20;

!\[\[113\_Craft2\_image0030.png]]

&#x20;

Then we can trigger the exploit.

!\[\[113\_Craft2\_image0031.png]]

&#x20;

{width="7.78125in" height="2.7291666666666665in"}

&#x20;

Flag:

!\[\[113\_Craft2\_image0033.png]]

&#x20;

Always Remember wertrigger exists!

&#x20;
