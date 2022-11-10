# Nukem

Nukem

Wednesday, 23 March 2022

12:00 pm

These are the ports.

!\[\[34\_Nukem\_image001.png]]

&#x20;

From here, a detailed nmap scan reveals this:

!\[\[34\_Nukem\_image002.png]]

These are the possible ports, and it seems that port 80 is a wp website!

&#x20;

Wpscan with the API token reveals a ton of vulnerabilities, most of which I don't know how to exploit.

I started with the plugins, because I thought they were the ones to exploit. There are loads of vulnerabilities regarding this tutor plugin installed on the website.

!\[\[34\_Nukem\_image003.png]]

This looked like the worst vulnerability.

&#x20;

!\[\[34\_Nukem\_image004.png]]

&#x20;

!\[\[34\_Nukem\_image005.png]]

&#x20;

Before running this, we can change the payload that is sent with the exploit.

!\[\[34\_Nukem\_image006.png]]

&#x20;

!\[\[34\_Nukem\_image007.png]]

Now, when we run it, it should give us something to use.

&#x20;

!\[\[34\_Nukem\_image008.png]]

&#x20;

Great, we have RCE!

!\[\[34\_Nukem\_image009.png]]

Now we have a curl shell.

&#x20;

Seems that netcat shells don't really work here.

Tried a bunch of ports before giving up on shells from that. Even my port 21 does not work, sad.

&#x20;

Anyways from this shell, we can read the flag. I grabbed that first before going back to think about how I was gonna shell in.

Finally landed on a python shell on port 80.

&#x20;

!\[\[34\_Nukem\_image0010.png]]

&#x20;

!\[\[34\_Nukem\_image0011.png]]

&#x20;

Ran a linpeas on this to check on stuff for me.

&#x20;

!\[\[34\_Nukem\_image0012.png]]

Dosbox.

&#x20;

Interestingly, there's this one port 5901, which is for VNC.

&#x20;

!\[\[34\_Nukem\_image0013.png]]

Combined with dosbox, it seems this is the way to go.

&#x20;

One of the next things to check is for some kind of password for the user commander. We can find it in the /srv/html/wp-config.php file as usual.

&#x20;

!\[\[34\_Nukem\_image0014.png]]

&#x20;

!\[\[34\_Nukem\_image0015.png]]

Now, we can write in our public key into the authorized\_keys and do some SSH port tunnelling.

This port 5901 is not accessible to us normally, but with port tunnelling it is.

&#x20;

!\[\[34\_Nukem\_image0016.png]]

After a little bit of troubleshooting, this worked.

&#x20;

We now have a terminal access to the user.

&#x20;

From here, we can just use this:

&#x20;

!\[\[34\_Nukem\_image0017.png]]

This would allow us to properly create a graphic shell in Z:/

&#x20;

We can change disk using C:

&#x20;

Then, we can access the root flag as root.

&#x20;

{width="10.197916666666666in" height="6.770833333333333in"}

&#x20;

However, this is not enough for OSCP (I think). I want to get a root shell still!

Using the shell, we can (painfully) echo stuff into the /etc/passwd file.

{width="6.458333333333333in" height="0.59375in"}

&#x20;

This would make it such that our user here has root access. From there SU and done! Wonderful box.

&#x20;
