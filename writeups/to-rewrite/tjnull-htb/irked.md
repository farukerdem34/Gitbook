# Irked

Irked

Thursday, 3 February 2022

7:07 pm

This is a Linux machine.

My IP address is 10.10.16.5.

Target IP is 10.10.10.117.

&#x20;

Nmap Scan up first.

!\[\[13\_Irked\_image001.png]]

SSH is open! Apache httpd 2.4.10 does not seem to have much weaknesses that allow RCE and stuff. Next is all the RPC ports, like port 111. I decided to do a quick enumeration of this port to see what's up. This port provides information between Unix based systems, allowing us to fingerprint the OS and obtain any other form of information possible. Picked up the fact that the hostname is irked.htb.

&#x20;

!\[\[13\_Irked\_image002.png]]

&#x20;

This shows the versions and stuff that are running on it, and we can see multiple different versions. Rather uninteresting as of now.

&#x20;

Moving to HTTP, we see this when we load up the HTTP server.

!\[\[13\_Irked\_image003.png]]

&#x20;

There's a hint there, which is always cool. IRC is almost working. IRC is a plain text protocol that usually runs on port 6667. In this case, it is running on port 6697 and 8067. The cheat sheet says that it avoids running the software with root privileges on those standard ports. This would perhaps indicate that the software is running on root privileges, and may be something we have to use to pivot.

&#x20;

Did a gobuster on the site in case, you never know! Found a /manual directory.

&#x20;

!\[\[13\_Irked\_image004.png]]

This /manual simply brings us straight to the Apache HTTP server documentation, something that is not very useful.

&#x20;

Anyways, we can try some banner grabbing on the thing, using **nc -vn \<target IP> 6697.** We can see that it does a DNS lookup of our IP address.

&#x20;

A bit more Google-fu shows us that in order to successfully infiltrate this is first add in the IP address to our hosts. We then need to pass in a nickname to connect to this port. We do the following and we suddenly get a connection.

!\[\[13\_Irked\_image005.png]]

&#x20;

&#x20;

From here we can see it's running the version **unreal 3.2.8.1**.

&#x20;

A quick searchsploit reveals there are some exploits for this!

!\[\[13\_Irked\_image006.png]]

&#x20;

Not gonna use MSF so let's try to find us an exploit that works. A bit of google led me [here](https://github.com/Ranger11Danger/UnrealIRCd-3.2.8.1-Backdoor).

This exploit worked! After running it I got an initial shell.

!\[\[13\_Irked\_image007.png]]

&#x20;

!\[\[13\_Irked\_image008.png]]

&#x20;

Spawned in a shell using python TTY payloads and we seem to be ircd. We have no privileges at all and we cannot get the user flag. From here, we need to pivot.

&#x20;

We spawn in the unreal3.2 configurations, so let's see if there are any configurations or whatever that is allowed in this thing. Anyways inside the directory of the user.txt, I did ls -la and there seems to be some form of hidden password and code..

!\[\[13\_Irked\_image009.png]]

&#x20;

Steg? Is there something hidden inside that image? There's one singular image on the web page...so I downloaded that. I used steghide to extract a pass.txt.

!\[\[13\_Irked\_image0010.png]]

Seems to be just a password. Using that to SSH into djmardov works! We now can capture that initial flag.

!\[\[13\_Irked\_image0011.png]]

&#x20;

Next step is to gain root access. Downloaded LinEnum.sh into the machine, and from there I went to see all the different processes that root could do.

!\[\[13\_Irked\_image0012.png]]

&#x20;

There's one portion of it that's weird. There's a /usr/bin/viewuser being executed by root.

!\[\[13\_Irked\_image0013.png]]

&#x20;

Seems out of the ordinary. When executed, it throws one error saying /tmp/listeners is not found.

&#x20;

!\[\[13\_Irked\_image0014.png]]

When testing, I put a string into that bit. Root cannot execute it at all, and it says permission denied. I made it executable and from LinEnum.sh, and I echoed in a root command and that gave me the shell!

!\[\[13\_Irked\_image0015.png]]

&#x20;

Done.

&#x20;

IRC is a tricky little thing. I knew earlier that there were exploits, but I did not know what version. Google-fu saved me on this one. Ls -la is another life saver!

&#x20;

Lazy to write security lessons learnt.
