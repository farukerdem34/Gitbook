# Timelapse (HTB) (WIP)

Timelapse (HTB) (WIP)

Tuesday, 29 March 2022

10:36 am

This is a Windows AD machine.

&#x20;

Always start with the default enum4linux to enumerate out the SMB shares and stuff.

!\[\[16\_Timelapse (HTB) (WIP)\_image001.png]]

By enumerating the smb as the username 'guest' we are able to get these out.

&#x20;

{width="10.447916666666666in" height="2.96875in"}

&#x20;

Connecting to these would reveals some extra details.

!\[\[16\_Timelapse (HTB) (WIP)\_image003.png]]

&#x20;

There are a few things in this:

!\[\[16\_Timelapse (HTB) (WIP)\_image004.png]]

&#x20;

We can see that there is a zip file, probably password protected if it's a winrm backup and some LAPS data stuffs. Not that I can open it either because it's in Microsoft Word and MSI formats.

&#x20;

Download all of these files onto our device.

{width="5.84375in" height="1.0in"}

&#x20;

As suspected, this is something that has been password protected.

&#x20;

{width="8.21875in" height="2.65625in"}

Simple cracking works on this to get a pfx file out.

&#x20;

This is actually a certificate and it contains the SSL certificate and private keys.

However, this is also password protected.

!\[\[16\_Timelapse (HTB) (WIP)\_image007.png]]

&#x20;

!\[\[16\_Timelapse (HTB) (WIP)\_image008.png]]

Found one user at least, and we likely need to winrm in as this user.

&#x20;

Pfx2john this thing, and we can get a password.

{width="10.46875in" height="2.7291666666666665in"}

Great!

From here, we can extract out the key.

!\[\[16\_Timelapse (HTB) (WIP)\_image0010.png]]

&#x20;

!\[\[16\_Timelapse (HTB) (WIP)\_image0011.png]]

&#x20;

Now that we have this private key, we can think about how to convert this to a user password and then evil-winrm in.

There are ways to import the certificate and get a scriptblock method of executing commands on the victim machine.

&#x20;

There are also ways of which we can directly access the AD Certificate Services, and use the crt and private key file that we extracted. Remember, this has got to do with some winrm\_backup, so we have to use that in order to get into the machine.

&#x20;

This is possible through the use of winrm\_shell.rn

!\[\[16\_Timelapse (HTB) (WIP)\_image0012.png]]

Once we have done this, it should look like so:

&#x20;

!\[\[16\_Timelapse (HTB) (WIP)\_image0013.png]]

&#x20;

Once we have this, we just have to execute the script accordingly.

&#x20;

!\[\[16\_Timelapse (HTB) (WIP)\_image0014.png]]

WE are in!

&#x20;

!\[\[16\_Timelapse (HTB) (WIP)\_image0015.png]]

We can grab the user flag from this easily.

From there, we can grab winpeas onto the machine.

!\[\[16\_Timelapse (HTB) (WIP)\_image0016.png]]

There seems to be another service user here.

&#x20;

!\[\[16\_Timelapse (HTB) (WIP)\_image0017.png]]

&#x20;

We are also part of the development group here.

Ran a linpeas to check on some stuff for me, and I found that there was this little powershell history text file here.

&#x20;

!\[\[16\_Timelapse (HTB) (WIP)\_image0018.png]]

&#x20;

Interesting! From here, we can go ahead and try to read that little bit...because normally this is not present on the machine.

Having it here is the interesting part.

!\[\[16\_Timelapse (HTB) (WIP)\_image0019.png]]

&#x20;

Reading this, there seems to be some kind of password here within the string that is used to execute some script blocks.

From here, we seem to be able to execute commands as the other user, as we are unable to win-rm in as him.

&#x20;

So in this case, we can use the scriptblocks to execute the commands as this user, and from there we can think about how to gain a reverse shell back as the user.

&#x20;

!\[\[16\_Timelapse (HTB) (WIP)\_image0020.png]]

&#x20;

Now, we can grab nc64.exe from our machine and then connect back to a listener port.

!\[\[16\_Timelapse (HTB) (WIP)\_image0021.png]]

Great! Now we are this user.

&#x20;

First thing I noticed was that this user was part of this global group:

!\[\[16\_Timelapse (HTB) (WIP)\_image0022.png]]

&#x20;

We are LAPS readers, which means we could potentially dump out the password of the administrator or something.

Because we are the LAPS readers, we can just read the password using some powershell:

&#x20;

!\[\[16\_Timelapse (HTB) (WIP)\_image0023.png]]

&#x20;

!\[\[16\_Timelapse (HTB) (WIP)\_image0024.png]]

&#x20;

Now that we have the password of the administrator, it is just a simple manner of either executing the scriptblocks again or just win-rm in.

&#x20;

We can actually reuse the method of which we execute commands as the svc\_deploy user and get an RCE as the administrator.

&#x20;

!\[\[16\_Timelapse (HTB) (WIP)\_image0025.png]]

&#x20;

!\[\[16\_Timelapse (HTB) (WIP)\_image0026.png]]

&#x20;

!\[\[16\_Timelapse (HTB) (WIP)\_image0027.png]]

&#x20;

!\[\[16\_Timelapse (HTB) (WIP)\_image0028.png]]

&#x20;

Pwned!

What a fun box. Wasn't that hard, but it did take a while before I found all the right tools.

&#x20;

&#x20;
