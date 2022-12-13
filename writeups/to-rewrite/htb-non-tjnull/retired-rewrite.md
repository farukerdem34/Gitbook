# Retired (NX + ASLR Bypass with ROP)

Retired (Fucking hard)

Thursday, June 23, 2022

7:54 AM

Nmap scan:

!\[\[30\_Retired (Fucking hard)\_image001.png]]

&#x20;

&#x20;

Port 80:

!\[\[30\_Retired (Fucking hard)\_image002.png]]

&#x20;

Page parameter is vulnerable to RFI.

This does not execute PHP code through.

&#x20;

Within the website, there is mention of this ostrich platform:

!\[\[30\_Retired (Fucking hard)\_image003.png]]

&#x20;

LFI:

We can find some useful stuff when we use php filters.

!\[\[30\_Retired (Fucking hard)\_image004.png]]

&#x20;

So there are users called vagrant and dev, presumably we are running as vagrant or www-data, because we can't read anything within the dev user directory.

&#x20;

We can access the /proc file through. So first, let's look at the processes that are running.

!\[\[30\_Retired (Fucking hard)\_image005.png]]

&#x20;

Interesting.

So this box is the classic buffer overflow ROP chain thingy, which would be going from LFI to RCE just using LFI...which is tough.

&#x20;

Let's take a look at the processes that the machine is running.

!\[\[30\_Retired (Fucking hard)\_image006.png]]

&#x20;

Most of these processes are pretty boring, except for one.

!\[\[30\_Retired (Fucking hard)\_image007.png]]

&#x20;

So there is this one binary running.

We can find out more about this binary through this link:

!\[\[30\_Retired (Fucking hard)\_image008.png]]

&#x20;

Then from there, we can download the binary.

!\[\[30\_Retired (Fucking hard)\_image009.png]]

&#x20;

!\[\[30\_Retired (Fucking hard)\_image0010.png]]

&#x20;

When running this, we can see that it requires a port to bind to:

!\[\[30\_Retired (Fucking hard)\_image0011.png]]

&#x20;

Interesting...this looks like a buffer overflow kind of thing.

{width="4.333333333333333in" height="1.1875in"}

&#x20;

Now, we can try to attach the some process to this elf file.

Lovely, I wanted to do this for long.

&#x20;

We'll be using GDB-Peda for this:

!\[\[30\_Retired (Fucking hard)\_image0013.png]]

&#x20;

So what this means is:

* CANARY is disabled, and hence there is no first line of defence
* FORTIFY is also disabled, and hence there are no checks against the functions used to protect this binary from BOF
* NS is enabled, and hence the program **cannot execute shell code**. This means that potentially, ret2libc or ROP chain is needed instead of generating shellcode.
* PIE means ASLR is enabled, which means we need to keep a script running...
* RELRO is FULL, meaning that READ ONLY RELOCATION is set to FULL of the entire program, meaning the writable storage area is minimal.

&#x20;

Right, so with this, we can attempt ret2libc or ROP chaining.

Return oriented programming is a pain in the ass.

&#x20;

Offset:

!\[\[30\_Retired (Fucking hard)\_image0014.png]]

&#x20;

Then we can crash this thing:

{width="11.34375in" height="1.7083333333333333in"}

&#x20;

Ok. Let's attempt to google about what can be potentially done here.

&#x20;

We know that we have a unique vulnerable binary to use, and we just need to find out what can be done with it.

&#x20;

&#x20;

&#x20;

Shell:

{width="4.28125in" height="0.5416666666666666in"}

&#x20;

{width="6.677083333333333in" height="1.34375in"}

&#x20;

Firstly, there are some weird zip files located within this.

!\[\[30\_Retired (Fucking hard)\_image0018.png]]

&#x20;

We can unzip this and see that it is indeed backups of the /var/www/html files.

So there's something like a scheduled task going on regarding these zip files, because they seem to be labelled with a time I think.

&#x20;

These scripts seem to run by any variation regarding the /var/www/html files, so when we edit it a bit, we can see that it re backs it up.

!\[\[30\_Retired (Fucking hard)\_image0019.png]]

&#x20;

The exploit is simple, we can create a symlink to the user's ssh keys, and it seems that the user in this case is dev.

!\[\[30\_Retired (Fucking hard)\_image0020.png]]

&#x20;

Exploit:

!\[\[30\_Retired (Fucking hard)\_image0021.png]]

&#x20;

!\[\[30\_Retired (Fucking hard)\_image0022.png]]

&#x20;

Cool.

&#x20;

Flag:

!\[\[30\_Retired (Fucking hard)\_image0023.png]]

&#x20;

PE:

I ran a linpeas with this user's elevated privileges.

{width="7.75in" height="4.09375in"}

&#x20;

It seems that we can write these, but I don't really know how to exploit it.

WIP
