# Jeeves

Jeeves

Friday, 11 March 2022

8:55 pm

This is a Windows machine.

The target IP is 10.10.10.63.

My IP is 10.10.16.9.

&#x20;

Enumeration time!

This is not an AD machine.

!\[\[76\_Jeeves\_image001.png]]

&#x20;

!\[\[76\_Jeeves\_image002.png]]

&#x20;

SMB:

Nothing much as it does not allow for null sessions.

&#x20;

Website:

!\[\[76\_Jeeves\_image003.png]]

This is a Jeeves kind of server. Googling anything within that search bar returns this:

!\[\[76\_Jeeves\_image004.png]]

&#x20;

I suppose this is some kind of version disclosure.

&#x20;

Port 50000:

!\[\[76\_Jeeves\_image005.png]]

&#x20;

Directory scans on port 80 revealed nothing.

&#x20;

Directory scan on port 50000 reveals this.

!\[\[76\_Jeeves\_image006.png]]

&#x20;

What be this directory here?

&#x20;

Going to it reveals a Jenkins server!

!\[\[76\_Jeeves\_image007.png]]

There's a version number there, and we're going to searchsploit it.

&#x20;

!\[\[76\_Jeeves\_image008.png]]

Of which it does not reveal anything.

&#x20;

Googling around for Jenkins exploits, we can find that we can load this script console page, of which we can probably execute code from here to gain a reverse shell or something.

&#x20;

!\[\[76\_Jeeves\_image009.png]]

&#x20;

Let's take a payload from PayloadAllTheThings.

&#x20;

!\[\[76\_Jeeves\_image0010.png]]

Run it and we catch a shell on a listener port.

!\[\[76\_Jeeves\_image0011.png]]

Grab the user flag.

&#x20;

Doing a whoami, I can see that we are this particular user.

!\[\[76\_Jeeves\_image0012.png]]

&#x20;

Right, let's check our privileges.

!\[\[76\_Jeeves\_image0013.png]]

Interesting, we are allowed to ImperonsatePrivilege.

&#x20;

Got WinPeas here, through powershell.

Here are the interesting parts:

&#x20;

!\[\[76\_Jeeves\_image0014.png]]

&#x20;

{width="9.34375in" height="0.8541666666666666in"}

&#x20;

!\[\[76\_Jeeves\_image0016.png]]

&#x20;

!\[\[76\_Jeeves\_image0017.png]]

&#x20;

!\[\[76\_Jeeves\_image0018.png]]

&#x20;

!\[\[76\_Jeeves\_image0019.png]]

&#x20;

The most interesting part was that CEH.kdbx file, the rest are kind of meh.

!\[\[76\_Jeeves\_image0020.png]]

Best way to transfer was using nc Imo, so let's get NC into this machine.

&#x20;

!\[\[76\_Jeeves\_image0021.png]]

&#x20;

Ran a keepass2john to convert it, followed by the bruteforce.

&#x20;

{width="9.90625in" height="4.3125in"}

Cool!

Now use Kpcli to log in.

&#x20;

!\[\[76\_Jeeves\_image0023.png]]

Let's look around this.

There's one group, and within that there are different types of entries.

&#x20;

!\[\[76\_Jeeves\_image0024.png]]

Looks like there's a secret kind of port there, and some kind of Jenkins admin username.

&#x20;

Let's view some of them:

!\[\[76\_Jeeves\_image0025.png]]

&#x20;

!\[\[76\_Jeeves\_image0026.png]]

&#x20;

!\[\[76\_Jeeves\_image0027.png]]

Looking at the first one, it seems to be a NTLM hash.

&#x20;

Perhaps we can try a pass the hash attack on this machine.

Looking around for pass the hash tools, we can use this one:

!\[\[76\_Jeeves\_image0028.png]]

&#x20;

Aight.

{width="9.729166666666666in" height="3.0416666666666665in"}

Cool!

But!

&#x20;

!\[\[76\_Jeeves\_image0030.png]]

Look deeper...Hmm...

Perhaps the flag is within the Windows data stream.

&#x20;

Let's try to view it.

!\[\[76\_Jeeves\_image0031.png]]

There's our flag, and this works because of the fact that the attribute of this file is hidden or something. So basically, our flag is within the $DATA stream, which is the data attribute of the file. The text from hm.txt would only contain the primary data stream of the file. Any other stream is considered alternate, and hence this stream is hidden in another stream.

&#x20;

One command would read it all.

!\[\[76\_Jeeves\_image0032.png]]

Done!

&#x20;

&#x20;
