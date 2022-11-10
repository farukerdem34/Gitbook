# WebCal

WebCal

Wednesday, 16 March 2022

5:34 pm

The target IP is 192.168.61.37.

My IP is 192.168.49.61.

&#x20;

!\[\[08\_WebCal\_image001.png]]

&#x20;

&#x20;

The FTP port is a little effy, we can't connect to it.

&#x20;

Website:

!\[\[08\_WebCal\_image002.png]]

There's a hint of an old website, so likely there's something about it.

&#x20;

The DNS port is also a bit effy.

!\[\[08\_WebCal\_image003.png]]

Port 21 is not FTP, it's something else.

&#x20;

When connected to the SMTP server, it seems that we are able to get out a domain name.

!\[\[08\_WebCal\_image004.png]]

Add this to our usual hosts file, and we can start enumerating it.

Started with vhosts, because there could be subdomains for this.

&#x20;

Tried a different wordlist for directory enumeration too, and this popped up with something.

!\[\[08\_WebCal\_image005.png]]

&#x20;

This was interesting, as it fit the title so I knew this had to be the way.

I ran another directory scan on this thing, and found loads of directories.

!\[\[08\_WebCal\_image006.png]]

All of it seems to redirect us back to the original login.php too and there are some that don't.

&#x20;

&#x20;

!\[\[08\_WebCal\_image007.png]]

Simple login with a version at the bottom.

&#x20;

!\[\[08\_WebCal\_image008.png]]

&#x20;

Seems like none of the versions fit possible exploits.

!\[\[08\_WebCal\_image009.png]]

&#x20;

Let's just try the 1.2.4 version, which works.

!\[\[08\_WebCal\_image0010.png]]

&#x20;

!\[\[08\_WebCal\_image0011.png]]

I get some form of shell, despite not being logged in.

Within this, there are loads of PHP files, presumably that of the web root.

{width="5.4375in" height="1.8645833333333333in"}

Within the config.php file, we see this.

&#x20;

Since this is a shell, I could technically place some form of command php here, and then execute code to give me a reverse shell.

&#x20;

From here, I did this command to search all files for passwords.

!\[\[08\_WebCal\_image0013.png]]

&#x20;

There were a few files with passwords.

!\[\[08\_WebCal\_image0014.png]]

The hash cannot be cracked, but the db\_password there was interesting.

&#x20;

From here, I easily got a cmd.php shell into the machine using wget.

!\[\[08\_WebCal\_image0015.png]]

&#x20;

!\[\[08\_WebCal\_image0016.png]]

Great. We have achieved persistence easily.

&#x20;

So from here, execute a quick bash shell and then grab a reverse shell on the listener port.

!\[\[08\_WebCal\_image0017.png]]

&#x20;

Within the /home directory was the local.txt

!\[\[08\_WebCal\_image0018.png]]

I ran a linpeas on this, as the usual directories of opt and stuff had nothing. I also downloaded pspy in case there were cron jobs going on.

&#x20;

&#x20;

!\[\[08\_WebCal\_image0019.png]]

There were a few exploits for this, and all of them seem to be some sort of privilege escalation thing.

&#x20;

But anyways, before that, let's take a look at the MySQL Server.

!\[\[08\_WebCal\_image0020.png]]

&#x20;

!\[\[08\_WebCal\_image0021.png]]

&#x20;

!\[\[08\_WebCal\_image0022.png]]

Within the webcal\_user table, there was the administrator password.

!\[\[08\_WebCal\_image0023.png]]

&#x20;

This hash was useless in the end.

&#x20;

Going back to the usual linpaes output, I tried some of the exploits that were available.

Tried cowroot and xorg, and went to the third one, which was memodipper.

Compiled it on my machine and then uploaded it onto the target machine.

!\[\[08\_WebCal\_image0024.png]]

&#x20;

Ran it, and rooted.

&#x20;

!\[\[08\_WebCal\_image0025.png]]

&#x20;

!\[\[08\_WebCal\_image0026.png]]

&#x20;
