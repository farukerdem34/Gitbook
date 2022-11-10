# Bounty

Bounty

Tuesday, 8 February 2022

1:19 pm

This is a Windows machine.

The target IP is 10.10.10.93.

My IP is 10.10.16.5.

&#x20;

Enumeration. Take note that we need to use our Vbscript thing, so we keep that in mind.

&#x20;

!\[\[25\_Bounty\_image001.png]]

&#x20;

The web server just displays Merlin the wizard, lol.

Looks like we have to run gobuster in order to try and fuzz something.

IIS 7.5 does not have vulnerabilities that I'm particularly interested in.

Ton of errors though, really dumb.

!\[\[25\_Bounty\_image002.png]]

&#x20;

Unsure what this entails for our exploits. There must be something I'm missing.

&#x20;

Ran more directory scans to try and see what else we can find. Running other directory finders, we can see that there is an aspnet\_client directory, which would mean any kind of backdoors we do we would need to use aspx.

&#x20;

!\[\[25\_Bounty\_image003.png]]

&#x20;

Managed to find a transfer.aspx as well, after running a dozen scans.

&#x20;

This is an upload file kind of situation here.

&#x20;

!\[\[25\_Bounty\_image004.png]]

&#x20;

It is likely that the files will be uploaded onto the Uploadedfiles folder within the present directories. I generated a quick .aspx folder to see what I can do with it.

&#x20;

Unfortunately, there's a kind of file WAF.

!\[\[25\_Bounty\_image005.png]]

&#x20;

Took a look at burpsuite, so let's see what we can do with this.

{width="5.71875in" height="2.5104166666666665in"}

&#x20;

We can use the null byte trick from Portswigger labs (thank god) to upload it.

Unfortunately, there seems to be a failure to execute the file.

!\[\[25\_Bounty\_image007.png]]

&#x20;

Looks like if we can't get the file to execute, we might need to use PowerShell instead, which would instantly get us a shell through running it on the device itself.

&#x20;

Took one from nishang, useful little directory.

Append this line to the end of the file.

!\[\[25\_Bounty\_image008.png]]

&#x20;

But this does not work.

&#x20;

I was running out of ideas, and resorted to the solution for the user shell.

&#x20;

Turns out, for IIS servers there is a web.config file that contains the settings and data for web apps on these servers. However, this file can include ASP code to get stuff to run.

&#x20;

So basically we can create a quick web.config file that is able to download the shell from us, and then execute it accordingly.

!\[\[25\_Bounty\_image009.png]]

&#x20;

This is the code that we can use, and this would download and execute the shell from us directly. This works, as it downloads the powershell script from us. It also executes it to give us a shell!

&#x20;

!\[\[25\_Bounty\_image0010.png]]

&#x20;

!\[\[25\_Bounty\_image0011.png]]

&#x20;

Cool! I googled how to use get-childitem because I knew that this was not a normal shell, but rather in PS.

PS is a bit slow, so I just slowly got the flag out.

!\[\[25\_Bounty\_image0012.png]]

We can use Get-Content to retrieve the flag.

&#x20;

Now it's time to Privilege escalate, and this is running a 64-bit Windows Server 2008 R2 Datacenter, with **no hotfixes.**

&#x20;

We can use sherlock to find out the different exploits that are possible. I chose the MS15-051 one, because I used it before.

&#x20;

Download this and another reverse shell onto the device.

&#x20;

!\[\[25\_Bounty\_image0013.png]]

&#x20;

Run it, and on the listener port we'll get the root shell.

!\[\[25\_Bounty\_image0014.png]]

&#x20;

This shell is a bit tedious for me. I looked at the answers, but from there I learnt that **sherlock.ps1** is a damn useful code. This would help sniff out which exploits are being used.

&#x20;

&#x20;
