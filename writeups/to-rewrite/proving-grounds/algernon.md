# Algernon

Algernon

Wednesday, 16 March 2022

2:55 pm

The target IP is 192.168.61.65.

My IP is 192.168.49.61.

&#x20;

!\[\[87\_Algernon\_image001.png]]

FTP server allows for anonymous logins.

{width="7.46875in" height="2.2604166666666665in"}

But there's no files we can browse there, so we can move on.

!\[\[87\_Algernon\_image003.png]]

&#x20;

Detailed namp scan reveals these:

!\[\[87\_Algernon\_image004.png]]

&#x20;

Port 80 is just a regular IIS server.

Port 9998 has a login page:

!\[\[87\_Algernon\_image005.png]]

&#x20;

There are a few vulnerabilities related to this machine.

!\[\[87\_Algernon\_image006.png]]

&#x20;

The last port open is MS .NET remoting services:

!\[\[87\_Algernon\_image007.png]]

This service has exploits regarding it.

&#x20;

I returned to smartermail and took the Build 6985 RCE, which had a py script and looked right.

I had no credentials or anything, so this was my best shot.

&#x20;

Looking at the exploit, this seems to send reverse shells to somewhere.

!\[\[87\_Algernon\_image008.png]]

It seems to do so using port 17001, which happened to be open. Why not try this?

&#x20;

!\[\[87\_Algernon\_image009.png]]

Ran it, and I got a shell back.

!\[\[87\_Algernon\_image0010.png]]

&#x20;

!\[\[87\_Algernon\_image0011.png]]

&#x20;

!\[\[87\_Algernon\_image0012.png]]

As root too.

&#x20;

!\[\[87\_Algernon\_image0013.png]]

&#x20;
