# ClamAV

ClamAV

Wednesday, 16 March 2022

2:20 pm

The machine IP is 192.168.61.42.

My IP is 192.168.49.61.

&#x20;

Enumerate this thing.

!\[\[06\_ClamAV\_image001.png]]

There's a fun little teaser on the HTTP port:

!\[\[06\_ClamAV\_image002.png]]

&#x20;

!\[\[06\_ClamAV\_image003.png]]

&#x20;

This could be a password, so I kept it.

!\[\[06\_ClamAV\_image004.png]]

Interestingly, there were 2 SSH ports...which was the first time I saw such a thing.

There seemed to be quite a few pitfalls here.

&#x20;

Port 80 offered nothing.

Port 25 was not able to be connected to.

SNMP is not useful as well.

&#x20;

I went to search for clamAV exploits instead.

!\[\[06\_ClamAV\_image005.png]]

Interesting, there were a few. The last one wasn't MSF so I installed that one to my machine.

&#x20;

{width="12.385416666666666in" height="5.71875in"}

This script here seems to get us a root shell or something.

Based on my understanding, it seems to open up the port 31337 or something.

!\[\[06\_ClamAV\_image007.png]]

This was initially closed.

&#x20;

{width="11.010416666666666in" height="5.145833333333333in"}

After a while this happened. It looks like the exploit worked properly. What this should do is enabled the port and then drop a shell into some port.

Now, when googling about this exploit, what was supposed to happen was that the port 31337 would open and we can nc into it.

&#x20;

!\[\[06\_ClamAV\_image009.png]]

&#x20;

!\[\[06\_ClamAV\_image0010.png]]

&#x20;

&#x20;

&#x20;
