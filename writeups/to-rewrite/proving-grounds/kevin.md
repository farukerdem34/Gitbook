# Kevin

Kevin

Wednesday, 16 March 2022

1:52 pm

The machine IP is 192.168.61.45.

My IP is 192.168.49.61.

&#x20;

!\[\[86\_Kevin\_image001.png]]

These are the boxes for this server.

&#x20;

Enum4linux does not reveal anything interesting.

Moving onto detailed Nmap scan.

!\[\[86\_Kevin\_image002.png]]

&#x20;

Website:

&#x20;

!\[\[86\_Kevin\_image003.png]]

Take note this is an .asp server, and this is some login page.

&#x20;

Default credentials of admin:admin get us in here.

!\[\[86\_Kevin\_image004.png]]

&#x20;

There seem to be a few exploits for this, regarding a BOF. For now, we will try to determine the version and stuff.

&#x20;

!\[\[86\_Kevin\_image005.png]]

&#x20;

I took the third one to try as it was the only python script :D.

&#x20;

Ran it without configuration first, and it works:

!\[\[86\_Kevin\_image006.png]]

&#x20;

Change the payload:

!\[\[86\_Kevin\_image007.png]]

&#x20;

!\[\[86\_Kevin\_image008.png]]

Run it, and we're in.

!\[\[86\_Kevin\_image009.png]]

&#x20;

!\[\[86\_Kevin\_image0010.png]]

&#x20;
