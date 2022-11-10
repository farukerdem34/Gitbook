# Sniper

Sniper

Sunday, June 19, 2022

9:40 PM

Nmap scan:

!\[\[25\_Sniper\_image001.png]]

&#x20;

!\[\[25\_Sniper\_image002.png]]

&#x20;

Port 80:

!\[\[25\_Sniper\_image003.png]]

&#x20;

Within this page, there are other links to other places.

At the end, there's this user portal that we see:

!\[\[25\_Sniper\_image004.png]]

&#x20;

&#x20;

This would bring us to another page:

!\[\[25\_Sniper\_image005.png]]

&#x20;

Creating fake user:

!\[\[25\_Sniper\_image006.png]]

&#x20;

When signed in, this is all we view.

!\[\[25\_Sniper\_image007.png]]

&#x20;

Not much here.

&#x20;

Blog:

!\[\[25\_Sniper\_image008.png]]

&#x20;

!\[\[25\_Sniper\_image009.png]]

&#x20;

When viewing the language, what's interesting is that it is another page instead of just the title of the language.

!\[\[25\_Sniper\_image0010.png]]

&#x20;

This looks like it could be vulnerable to some LFI.

We can test this thing out.

&#x20;

Apart from everything else that we already saw, this seems to be the most suspicious place of which there are possible links.

&#x20;

Wfuzz scan:

!\[\[25\_Sniper\_image0011.png]]

&#x20;

Wordlist:

[https://github.com/carlospolop/Auto\_Wordlists/blob/main/wordlists/file\_inclusion\_windows.txt](https://github.com/carlospolop/Auto\_Wordlists/blob/main/wordlists/file\_inclusion\_windows.txt)

&#x20;

Nothing was found from there.

When trying to view index.php, there was a hang:

!\[\[25\_Sniper\_image0012.png]]

&#x20;

I tried this thing from hacktricks, which was to try and steal NTLM hashes:

!\[\[25\_Sniper\_image0013.png]]

&#x20;

Starting responder:

{width="6.46875in" height="2.9375in"}

&#x20;

SMB Link used (IP Address changed):

!\[\[25\_Sniper\_image0015.png]]

&#x20;

Turns out my Responder was broken the whole time, and this works with SMBserver.py for some reason.

&#x20;

Anyways, with this we can still execute files over SMB to gain a shell.

This blocks out all other forms of files except for PHP files, so we can create a quick one here.

&#x20;

Shell:

!\[\[25\_Sniper\_image0016.png]]

&#x20;

{width="7.0in" height="2.2291666666666665in"}

&#x20;

PE:

!\[\[25\_Sniper\_image0018.png]]

&#x20;

There were some credentials found here.

&#x20;

Method 1:

Checking privileges:

!\[\[25\_Sniper\_image0019.png]]

&#x20;

Cool.

We can use printspoofer.

{width="7.8125in" height="4.3125in"}

&#x20;

!\[\[25\_Sniper\_image0021.png]]

&#x20;

But that would be too easy.

&#x20;

Method 2:

We had a password and perhaps we could play around with executing command blocks remotely.

Chris privileges:

!\[\[25\_Sniper\_image0022.png]]

&#x20;

He is a remote management user, meaning we can execute commands on his behalf as long as we have his password.

RCE as Chris:

!\[\[25\_Sniper\_image0023.png]]

&#x20;

Now, we can gain a reverse shell easily.

!\[\[25\_Sniper\_image0024.png]]

&#x20;

{width="6.78125in" height="2.2604166666666665in"}

&#x20;

Flag:

!\[\[25\_Sniper\_image0026.png]]

897c6a31a6cb40e6cc7c3ccb71657b52

&#x20;

PE2:

We can now access the C:\Docs folder.

!\[\[25\_Sniper\_image0027.png]]

&#x20;

!\[\[25\_Sniper\_image0028.png]]

&#x20;

Within the Downloads folder of this user, we can find this:

!\[\[25\_Sniper\_image0029.png]]

&#x20;

There is a chm file, which is basically a compiled HTML Help file that Microsoft uses for stuff.

We can transfer this one back to our file for some analysis.

!\[\[25\_Sniper\_image0030.png]]

&#x20;

!\[\[25\_Sniper\_image0031.png]]

&#x20;

We can use xchm to read this file.

!\[\[25\_Sniper\_image0032.png]]

&#x20;

!\[\[25\_Sniper\_image0033.png]]

&#x20;

Well that's nice.

Anyways, it's clear that we need to create some form of CHM shell or executable that would give us a shell.

We can use Out-CHM.ps1.

&#x20;

!\[\[25\_Sniper\_image0034.png]]

&#x20;

Unfortunately, the download of the executable needed is no longer around...

So this is the method, we have to create a .chm file and then put it there and it would execute.

&#x20;

Lazy to continue because we already root using PrintSpoofer which was the more...obvious solution.

Good machine through.

Awdawdawd345634563456435643564564654564564werwerwerwerwerwewerwerwer

&#x20;
