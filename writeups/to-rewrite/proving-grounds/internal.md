# Internal

Internal

Wednesday, 16 March 2022

12:10 pm

My IP is 192.168.49.73.

The target IP is 192.168.73.40

&#x20;

Running an Nmap scan here:

!\[\[84\_Internal\_image001.png]]

&#x20;

Seeing as that there are some SMB ports, I ran enum4linux too, which returned nothing.

SMBClient returned nothing too. So did SMBMap.

&#x20;

Detailed scan:

!\[\[84\_Internal\_image002.png]]

I see potential HTTP servers, and also a DNS port.

&#x20;

DNS ports tell me there has to be something regarding some form of ports.

I then ran some scans on the RDP port too.

&#x20;

!\[\[84\_Internal\_image003.png]]

&#x20;

!\[\[84\_Internal\_image004.png]]

&#x20;

Now, we have one vulnerability to work with.

Looking at some of the possible exploit scripts that are available on the web, this seems to be some sort of buffer overflow trick:

!\[\[84\_Internal\_image005.png]]

&#x20;

However, I was unable to get anything from this.

&#x20;

Moving on, since this RDP was pretty old, I wanted to test the SMB stuff too.

Ran this command:

!\[\[84\_Internal\_image006.png]]

&#x20;

Now, let's see what can happen.

!\[\[84\_Internal\_image007.png]]

There's another vulnerability.

&#x20;

Googled and found another exploit script that looks more promising.

&#x20;

!\[\[84\_Internal\_image008.png]]

&#x20;

First, let's generate another possible msfvenom shell.

&#x20;

{width="11.1875in" height="4.53125in"}

&#x20;

This almost works.

!\[\[84\_Internal\_image0010.png]]

&#x20;

!\[\[84\_Internal\_image0011.png]]

The connection works, but something is wrong with my code.

&#x20;

Perhaps we have to use MSF exploit handler to do this one, I changed the code to fit that easily.

Generated another one.

&#x20;

{width="9.8125in" height="2.6979166666666665in"}

&#x20;

So I found out my box crashed halfway through, which was why my exploits were not really working...

The boxes keep crashing for me.

&#x20;

Decided to go for MSF instead again, which always sucks.

Ran the exploit/windows/smb/ms09\_050\_smb2\_negotiate\_func\_index module, and just waited.

&#x20;

{width="9.104166666666666in" height="3.7291666666666665in"}

My IP changed yet again, which was annoying but not much of a problem.

&#x20;

{width="11.229166666666666in" height="4.46875in"}

Welp.

&#x20;

!\[\[84\_Internal\_image0015.png]]

&#x20;
