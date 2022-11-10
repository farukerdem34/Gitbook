# Pelican

Pelican

Tuesday, 15 March 2022

3:36 pm

This is a Linux machine.

The target IP is 192.168.73.98.

My IP is 192.168.49.73.

&#x20;

!\[\[02\_Pelican\_image001.png]]

Right, so we have these to work with.

&#x20;

Let's view the ports in HTTP.

Viewing port 8080 gives us nothing.

!\[\[02\_Pelican\_image002.png]]

&#x20;

Viewing port 8081 gives us this interesting ZooKeeper kind of webpage.

&#x20;

!\[\[02\_Pelican\_image003.png]]

&#x20;

Doing a searchsploit on both, it seems that these come up.

&#x20;

!\[\[02\_Pelican\_image004.png]]

Interestingly, there's this version in the corner:

!\[\[02\_Pelican\_image005.png]]

I was interested in that exhibitor exploit I saw, and downloaded it to my machine.

&#x20;

!\[\[02\_Pelican\_image006.png]]

&#x20;

Reading it, it seems that it would be able to execute commands based on certain versions of the ZooKeeper.

&#x20;

!\[\[02\_Pelican\_image007.png]]

&#x20;

So reading about the vulnerability, it seems that within the config tab of this web engine, there is this java.env script.

!\[\[02\_Pelican\_image008.png]]

Looking at it, it passes variables as $JVMFLAGS.

&#x20;

From here, we are able to append expressions and hence commands.

&#x20;

To test this, I used a simple command based on the exploit to ping my machine. I then set up a ICMP listener port.

!\[\[02\_Pelican\_image009.png]]

&#x20;

!\[\[02\_Pelican\_image0010.png]]

&#x20;

Afterwards, just press commit and commit all:

&#x20;

!\[\[02\_Pelican\_image0011.png]]

&#x20;

!\[\[02\_Pelican\_image0012.png]]

Then Click OK:

!\[\[02\_Pelican\_image0013.png]]

&#x20;

{width="10.072916666666666in" height="1.5625in"}

This was the Proof of Concept of the exploit.

&#x20;

Now, we can set up a listener netcat and wait for a shell to drop in. I'll be using the netcat command there, but there are other commands such as python or bash shells that can do the same.

&#x20;

!\[\[02\_Pelican\_image0015.png]]

&#x20;

!\[\[02\_Pelican\_image0016.png]]

There we go, we have some port.

&#x20;

Now I like to use spawn a TTY shell, and then upgrade my shell here so that I have auto-complete and also up and down arrow keys.

To do so, simply use this python script.

&#x20;

!\[\[02\_Pelican\_image0017.png]]

Now we have a simple prompt.

&#x20;

Then, simply run CTRL+Z and then stty raw -echo;fg and then hit enter twice.

&#x20;

{width="3.7604166666666665in" height="2.09375in"}

From here, we can just do bash -I to get a nice prompt.

{width="4.03125in" height="1.6458333333333333in"}

Great!

!\[\[02\_Pelican\_image0020.png]]

&#x20;

That's the POC for the flag, now we need to become root!

Now, from here, we can grab linpeas from our machine easily.

&#x20;

Set up a quick python server with the web root within my machine.

&#x20;

!\[\[02\_Pelican\_image0021.png]]

Yes, that's where I store all of my files.

&#x20;

From there, we can just wget linpeas.sh into the machine and then run it.

D

!\[\[02\_Pelican\_image0022.png]]

Run this script, and what it does for us is basically enumerate the machine for us. Again, while this is allowed, please do make sure that your manual enumeration skills are good too. Looking at the exploit, this does not seem like a hard box at all, so I just use this.

&#x20;

Anyways, linpeas would have highlighted some of the more obvious PE vectors as shown:

&#x20;

!\[\[02\_Pelican\_image0023.png]]

Now, this is an example. This sudo version is vulnerable. However, since this exploit came out quite recently, I do not think it is the intended solution.

&#x20;

!\[\[02\_Pelican\_image0024.png]]

There seems to be a script being run within a public directory that is able to run stuff. Notice that this is run my root, which means if we can inject some bash code into this script, we can easily just get a root reverse shell as root would execute it.

&#x20;

However, this would not work for us as @reboot is not a good sign, and it seems that I do not have the ability to reboot the machine.

Generally when exploiting services for OSCP or something, we would like to pick one that we can easily execute ourselves instead of one whereby we have to wait for the user to reset the machine for us.

&#x20;

Looking at the user privileges, we can see this:

!\[\[02\_Pelican\_image0025.png]]

Now this is definitely one PE vector. This is because the user charles, which we are logged in as right now, is able to execute gcore as root without a password. This is a classic case of misconfiguration when it comes to managing a machine.

&#x20;

Now, from here, we have our vector. Seeing that this is an SUID, I went to GTFOBins.github.io, which is basically a giant repository of all the different SUIDs privilege misconfigurations we can abuse to escape a shell.

&#x20;

!\[\[02\_Pelican\_image0026.png]]

&#x20;

!\[\[02\_Pelican\_image0027.png]]

&#x20;

!\[\[02\_Pelican\_image0028.png]]

Alright, it looks like we can read files through identifying the guid.

&#x20;

So first, we need to identify the processes that are on the machine, and preferably something that stores credentials, because we can easily exploit that.

&#x20;

{width="7.4375in" height="1.71875in"}

&#x20;

!\[\[02\_Pelican\_image0030.png]]

&#x20;

This looks like one that could work.

!\[\[02\_Pelican\_image0031.png]]

From here, the website says that we can use strings to filter down the content we get.

&#x20;

!\[\[02\_Pelican\_image0032.png]]

Within this, we can see this one thing:

&#x20;

!\[\[02\_Pelican\_image0033.png]]

From there, I tried SSH but it did not work. So the next option is to Su, which is switch user.

&#x20;

!\[\[02\_Pelican\_image0034.png]]

&#x20;

!\[\[02\_Pelican\_image0035.png]]

Pwned.

&#x20;

Recommendations:

1. Update the web engine! There's a serious RCE on there!
2. Always check privileges of your users, make sure that they are not sudoing anything.
3. Also, side note, but also having scripts run as above is still considered dangerous because of the fact that this could be exploited through writing into the crontab as root, which can lead to pivoting or something else.

&#x20;

&#x20;
