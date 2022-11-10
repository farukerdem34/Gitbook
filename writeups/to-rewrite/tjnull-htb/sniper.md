# Sniper

Sniper

Sunday, 13 March 2022

10:25 am

This is a Windows machine.

The target IP is 10.10.10.151.

My IP is 10.10.16.9.

&#x20;

Nmap this first.

!\[\[81\_Sniper\_image001.png]]

&#x20;

!\[\[81\_Sniper\_image002.png]]

&#x20;

Website:

!\[\[81\_Sniper\_image003.png]]

All the links don't work except one for the login page.

&#x20;

!\[\[81\_Sniper\_image004.png]]

Created a quick account.

&#x20;

When visited, it just shows this:

&#x20;

!\[\[81\_Sniper\_image005.png]]

So there's a new website or something.

&#x20;

There's also this blog accessible through the services link:

!\[\[81\_Sniper\_image006.png]]

&#x20;

Looking at the languages part, we can see this pops up when we request a different language:

!\[\[81\_Sniper\_image007.png]]

This could be abused for some file inclusion or directory traversal.

&#x20;

This could be exploited, as I tried visiting something like ../index.php, but it shows this page:

!\[\[81\_Sniper\_image008.png]]

This was interesting.

&#x20;

Anyways, the next step would be to check whether or not it could connect to us. (hint from forums!)

Tried some SMB stuff.

&#x20;

I had one file within SMB root, and I tried this one:

!\[\[81\_Sniper\_image009.png]]

This worked!

{width="15.260416666666666in" height="3.4375in"}

&#x20;

We can include a cmd php file followed by another parameter for the cmd. I did this:

!\[\[81\_Sniper\_image0011.png]]

&#x20;

!\[\[81\_Sniper\_image0012.png]]

Right, now we have RFI.

&#x20;

Download nc64.exe into the machine and then let's give us a reverse shell.

Checking the dir, we are able to see this:

!\[\[81\_Sniper\_image0013.png]]

Right, so we need to get nc there somehow.

&#x20;

Downloaded it in using powershell -c wget.

&#x20;

!\[\[81\_Sniper\_image0014.png]]

&#x20;

!\[\[81\_Sniper\_image0015.png]]

Right, so now set up a listener port and let's get to it.

&#x20;

Found out I was not allowed to put files there, and went hunting for some kind of files I can use.

Googled and found this one file

!\[\[81\_Sniper\_image0016.png]]

Got nc there and executed the shell.

&#x20;

&#x20;

&#x20;

!\[\[81\_Sniper\_image0017.png]]

Great.

&#x20;

Within the /inetpub/wwwroot/user file, there was one db.php, which contained some passwords:

!\[\[81\_Sniper\_image0018.png]]

&#x20;

Checking the users, there seems to be a Chris:

!\[\[81\_Sniper\_image0019.png]]

&#x20;

Checking the permissions on Chris, found out he was part of the Remote Management Use group.

&#x20;

!\[\[81\_Sniper\_image0020.png]]

Unfortunately evil-winrm does not work.

&#x20;

This password has to be something however.

&#x20;

Found this command online about how to change users:

!\[\[81\_Sniper\_image0021.png]]

Right, that's useful.

Went on hacktricks as well, and found a shorter version which would allow us to basically sudo stuff.

&#x20;

!\[\[81\_Sniper\_image0022.png]]

&#x20;

From here, we can test it and see that it works.

&#x20;

!\[\[81\_Sniper\_image0023.png]]

Cool.

&#x20;

Now we just need to get that nc.exe file and give back another reverse shell to us.

&#x20;

!\[\[81\_Sniper\_image0024.png]]

&#x20;

Within the downloads file of Chris, there is this instructions.chm folder.

&#x20;

D

!\[\[81\_Sniper\_image0025.png]]

.chm means compressed HTML or something.

&#x20;

I'm not entirely sure what this contains, but I was googling on how to use CHM to privilege escalate.

&#x20;

Was stuck here for a long ass time.

I found this online:

!\[\[81\_Sniper\_image0026.png]]

Alright, so this CHM file is indeed the answer.

&#x20;

Also found [this](https://deepsec.net/docs/Slides/2014/Client\_Side\_Attacks\_PowerShell\_Nikhil\_Mittal.pdf) PDF explaining to me how this can be done.

&#x20;

So we have to use Out-CHM.ps1 which would allow us to execute command using this CHM file. This would be executed as a cron job perhaps, and we could get a root shell from this.

&#x20;

Let's get it onto the machine:

!\[\[81\_Sniper\_image0027.png]]

Some cron job or firewall is deleting my file every so often...

This is definitely flagged as malicious.

&#x20;

Couldn't get it to work and resorted to a walkthrough.

&#x20;

So turns out I needed a Windows machine to transfer files over.

&#x20;

Won't be doing this for now, I haven't configured a Windows VM..

But the trick is to compile the malicious CHM file within Windows, and then import it to Linux. From there, we can upload it and get an administrator reverse shell.

Good to know that I need to configure a Windows VM.

&#x20;

&#x20;

&#x20;

&#x20;

&#x20;

&#x20;
