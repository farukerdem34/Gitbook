# Arctic

Arctic

Saturday, 12 March 2022

4:53 pm

This is a really slow Windows machine.

The target IP is 10.10.10.11.

My IP is 10.10.16.9.

&#x20;

Let's enumerate this thing.

Take note that this is a rather slow but simpler machine.

Something to do with file uploading...

!\[\[79\_Arctic\_image001.png]]

These are the ports, and most notably is port 8500.

&#x20;

{width="10.604166666666666in" height="2.71875in"}

These are the possibilities. Let's try to connect to it and see what happens from there.

Take note this takes a long time before anything really happens.

&#x20;

Take note this wait is very very painful, as it takes like minutes before it loads anything.

From here, I already determined that it is possibly a ColdFusion webserver, because I am able to connect to it but nothing happens.

&#x20;

!\[\[79\_Arctic\_image003.png]]

Coldfusion is actually a commercial web-application development platform. So from here, let's try to find some exploits for it.

&#x20;

{width="11.760416666666666in" height="7.989583333333333in"}

Plenty here, and judging from the difficulty of the box, I suspect it might be ColdFusion 8.

**Take note, do not do this. I'm only doing this because the box takes forever!**

&#x20;

!\[\[79\_Arctic\_image005.png]]

Change these parameters and judging the starting IP, this is definitely the right exploit.

Run it, and this would not require us to set up a listener port as this exploit does it all for us.

&#x20;

!\[\[79\_Arctic\_image006.png]]

&#x20;

Eventually we are greeted with a shell.

&#x20;

!\[\[79\_Arctic\_image007.png]]

&#x20;

Grab the user flag!

&#x20;

Now, let's check our privileges as this user.

!\[\[79\_Arctic\_image008.png]]

Let's check the system info as well.

&#x20;

{width="9.59375in" height="9.135416666666666in"}

&#x20;

I see that this has no hotfixes and there SeImpersonateToken thing is enabled. JuicyPotato might work here.

Googling around, found this windows exploit suggester tool, which was really useful indeed.

It's called wesng.py.

&#x20;

Loads of output.

&#x20;

!\[\[79\_Arctic\_image0010.png]]

Eventually, I found one that works well, called MS09-012 or CVE-2009-0079.

&#x20;

It's called Chimichurri.exe, ran it and it requires a listener port.

&#x20;

!\[\[79\_Arctic\_image0011.png]]

Well pwned.

&#x20;

&#x20;

&#x20;

&#x20;
