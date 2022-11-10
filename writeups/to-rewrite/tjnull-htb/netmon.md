# Netmon

Netmon

Sunday, 13 March 2022

12:10 pm

This is a Windows machine.

The target IP is 10.10.10.

My IP is 10.10.16.9.

&#x20;

Enumeration time. Apparently the user flag took 1 minute to capture...wow.

Enumerate this and let's move on.

&#x20;

!\[\[82\_Netmon\_image001.png]]

Aight.

Ran an enum4linux and went to the HTTP server while detailed Nmap was running.

&#x20;

Website:

!\[\[82\_Netmon\_image002.png]]

&#x20;

!\[\[82\_Netmon\_image003.png]]

Version here!

&#x20;

Well, this version is not vulnerable.

&#x20;

From here, I saw an FTP server, and tried the anonymous login trick.

&#x20;

!\[\[82\_Netmon\_image004.png]]

As you can see, the user flag takes 1 minute to capture...

&#x20;

Now moving around this space, I notice there's the whole directory of the PRTG Network monitor here.

Googling, it reveals that some config files can be found here.

!\[\[82\_Netmon\_image005.png]]

&#x20;

!\[\[82\_Netmon\_image006.png]]

Went into All Users and then found these.

Transferred the current Configurations files.

&#x20;

Within most of the folders, the Password has been redacted. However, in the .old.bak file, there was this:

!\[\[82\_Netmon\_image007.png]]

Great.

&#x20;

Still could not log in though. I tried lots of things but all didn't work. Based on a hint, turns out we just had to change the year to 2019. Well, pretty dumb.

&#x20;

Found an exploit online:

!\[\[82\_Netmon\_image008.png]]

This would work, but I prefer manual exploitation for this box.

&#x20;

Now, looking at a few more blogs, this [this](https://www.codewatch.org/blog/?p=453) was the best.

There's a vulnerability within the notifications tab, and when viewed, it seems to allow us to input some program through the Execute Program tab.

&#x20;

Found this on [github](https://github.com/chcx/PRTG-Network-Monitor-RCE).

&#x20;

Running this would create a new user with administrator privileges.

&#x20;

!\[\[82\_Netmon\_image009.png]]

&#x20;

Evil-winrm in.

!\[\[82\_Netmon\_image0010.png]]

&#x20;

Like I said, this user has admin privileges.

&#x20;

!\[\[82\_Netmon\_image0011.png]]

&#x20;
