# Resolute (HTB)

Resolute (HTB)

Monday, 28 March 2022

8:59 pm

This is a simple AD machine.

When running an enum4linux to check for nullsession details, a lot comes out.

&#x20;

We have some possible users here.

{width="5.28125in" height="6.34375in"}

&#x20;

{width="3.1041666666666665in" height="1.1458333333333333in"}

This is the domain.

&#x20;

Also, there's one interesting description here.

!\[\[15\_Resolute (HTB)\_image003.png]]

&#x20;

Get all of the users into one singular username file.

&#x20;

From here, because we have some valid credentials, I it to do a crackmapexec for all the users on SMB.

&#x20;

{width="10.197916666666666in" height="0.6979166666666666in"}

Right, we have some credentials.

&#x20;

With this, we can evil-winrm in.

&#x20;

!\[\[15\_Resolute (HTB)\_image005.png]]

&#x20;

There are 2 other users on this machine:

!\[\[15\_Resolute (HTB)\_image006.png]]

&#x20;

!\[\[15\_Resolute (HTB)\_image007.png]]

Net user ryan seems to be part of the Contractors group, while Melanie is only part of the Remote Management Users group.

&#x20;

Ran a winpeas on this machine to enumerate it further before carrying on to look at:

1. Registries
2. Group Policy stuff

&#x20;

If there is nothing else we can do, Bloodhound would work as well.

&#x20;

There are some hidden directories, and this one has this text file with details within it about something.

!\[\[15\_Resolute (HTB)\_image008.png]]

&#x20;

!\[\[15\_Resolute (HTB)\_image009.png]]

This looks like the service account password.

&#x20;

From there, I just tried the evil-winrm and it still worked despite ryan not being part of the other group.

!\[\[15\_Resolute (HTB)\_image0010.png]]

&#x20;

On ryan's desktop, there's a notes file.

!\[\[15\_Resolute (HTB)\_image0011.png]]

&#x20;

!\[\[15\_Resolute (HTB)\_image0012.png]]

&#x20;

I...don't know what we can do with this. But it just tells me there is some scheduled job ongoing that I could perhaps hijack and allow it to PE me to administrator.

&#x20;

I wanted to see the scheduled tasks that were on going, but it seems that my access is denied.

{width="12.135416666666666in" height="3.8229166666666665in"}

I did a whoami /all to find out more information about this service account, and I found out that ryan is part of the DNSadmins group.

&#x20;

!\[\[15\_Resolute (HTB)\_image0014.png]]

&#x20;

!\[\[15\_Resolute (HTB)\_image0015.png]]

&#x20;

This would mean I have control over some of the processes that are running on this device, allowing me to hijack DLLs or something.

Firstly I created a malicious DLL file here and started a listener

!\[\[15\_Resolute (HTB)\_image0016.png]]

&#x20;

Then start a smbserver with the root folder containing the malicious DLL.

!\[\[15\_Resolute (HTB)\_image0017.png]]

&#x20;

From there we need to use dnscmd to reset the DNS and stuff.

!\[\[15\_Resolute (HTB)\_image0018.png]]

Once we have done that, we have just confirmed that our malicious DLL got loaded.

!\[\[15\_Resolute (HTB)\_image0016.png]]

&#x20;

&#x20;

After that, just do sc.exe stop dns and sc.exe start dns, and we should get an admin shell on our listener port.

!\[\[15\_Resolute (HTB)\_image0019.png]]

&#x20;

{width="5.59375in" height="2.1041666666666665in"}

Pwned!

&#x20;
