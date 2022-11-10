# Antique

Antique

Monday, 28 March 2022

3:23 pm

This is another Linux machine with these ports that are open.

!\[\[06\_Antique\_image001.png]]

&#x20;

Interesting.

&#x20;

When we connect via NC, we get this HP JetDirect command line.

!\[\[06\_Antique\_image002.png]]

There's no way to brute force or check this one, so we should go back to Nmap and search for UDP ports.

Went to google a bit more about hacking printers, and came across this SNMP exploit.

&#x20;

!\[\[06\_Antique\_image003.png]]

&#x20;

We could potentially grab the password out.

&#x20;

Tried it and it works!

!\[\[06\_Antique\_image004.png]]

&#x20;

We get this string of numbers, and it seems to be hex according to the website.

!\[\[06\_Antique\_image005.png]]

We can sort of make out that there's the word 'password' in there.

!\[\[06\_Antique\_image006.png]]

Right, we have the password!

&#x20;

!\[\[06\_Antique\_image007.png]]

&#x20;

!\[\[06\_Antique\_image008.png]]

From this, we have RCE!

!\[\[06\_Antique\_image009.png]]

&#x20;

!\[\[06\_Antique\_image0010.png]]

&#x20;

From here, we can grab the user flag.

&#x20;

This user seems to be vulnerable to the sudo exploit, but let's not use that one.

!\[\[06\_Antique\_image0011.png]]

Interestingly, port 631 is open.

This is 100% a port tunnel thing, because we might need to pivot from this printer to the main device.

&#x20;

Using Socat as I do not have SSH, I was able to tunnel and forward traffic to port 9090.

&#x20;

{width="7.875in" height="1.6770833333333333in"}

&#x20;

When trying to connect to it, I noticed the version it was running.

&#x20;

!\[\[06\_Antique\_image0013.png]]

CUPS, and if they give a version, it has to be somehow vulnerable.

!\[\[06\_Antique\_image0014.png]]

&#x20;

Read the MSF code.

{width="10.791666666666666in" height="2.5729166666666665in"}

&#x20;

This exploit takes a file that we want to read and does this:

!\[\[06\_Antique\_image0016.png]]

It seems to take the ctl path and converts the ErrorLog to be the file we want to read. Afterwhich, it curls itself basically.

Simple enough, we just need to use cupsctl to convert the error log to the root flag, and then curl that.

&#x20;

!\[\[06\_Antique\_image0017.png]]

I do not think there is a way for us to read the password for the root file, and even if there is, the hash might not be crack able.

&#x20;

&#x20;
