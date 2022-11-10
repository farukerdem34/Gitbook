# GoodGames

GoodGames

Monday, 28 March 2022

2:39 pm

A Windows machine with a singular port 80 that is open.

Scanning it reveals that this is indeed a Python server, which we may need to know for later.

&#x20;

!\[\[05\_GoodGames\_image001.png]]

&#x20;

!\[\[05\_GoodGames\_image002.png]]

This be the website, and there's a log in here.

&#x20;

!\[\[05\_GoodGames\_image003.png]]

Upon seeing this, I tested it against some SQL Injections.

{width="12.6875in" height="0.625in"}

Looks good.

&#x20;

I was unable to dump the database, hence tried the old trick of logging in with ' or 1=1 --

&#x20;

!\[\[05\_GoodGames\_image005.png]]

&#x20;

!\[\[05\_GoodGames\_image006.png]]

Cool. We are in.

&#x20;

From here, it seems that we are admin@goodgames.htb. We can first change the password to something easier.

When looking at the settings, we get this error.

!\[\[05\_GoodGames\_image007.png]]

&#x20;

Simply fixed.

When clicking it, we get redirected to another login page.

&#x20;

!\[\[05\_GoodGames\_image008.png]]

Saw this and tried the same SQL Injection trick, but it does not work.

&#x20;

I decided to run a hydra to test it. It took too long however, so I just tried some random admin passwords from rock you and found out that superadministrator was the password.

&#x20;

!\[\[05\_GoodGames\_image009.png]]

&#x20;

Interestingly, when viewing the profile, we can update our full name and it appears on the right.

&#x20;

!\[\[05\_GoodGames\_image0010.png]]

&#x20;

The fact that it prints out on the page again shows me that it might be an SSTI or something. I tried the usual payloads.

&#x20;

!\[\[05\_GoodGames\_image0011.png]]

&#x20;

!\[\[05\_GoodGames\_image0012.png]]

We now have SSTI!

&#x20;

This is a Python based server based on the domain we scanned earlier, so we should look at Python or Flask based SSTI payloads.

!\[\[05\_GoodGames\_image0013.png]]

Entered \{{config.items()} to confirm this is a Flask SSTI.

!\[\[05\_GoodGames\_image0014.png]]

&#x20;

!\[\[05\_GoodGames\_image0015.png]]

&#x20;

This payload provides for the command injection we need. Now, we just need to have an RCE to reverse shell back to us.

&#x20;

We can download shells using this payload here.

!\[\[05\_GoodGames\_image0016.png]]

&#x20;

!\[\[05\_GoodGames\_image0017.png]]

This works out well, and we can now try to get a reverse shell from the machine.

!\[\[05\_GoodGames\_image0018.png]]

&#x20;

Looks like we're root inside a container here. Gotta escape somehow.

&#x20;

{width="4.802083333333333in" height="1.4375in"}

&#x20;

!\[\[05\_GoodGames\_image0020.png]]

SDA 1 looks very interesting.

!\[\[05\_GoodGames\_image0021.png]]

&#x20;

Perhaps this is the mount point.

&#x20;

!\[\[05\_GoodGames\_image0022.png]]

There are some IDs here too.

&#x20;

I think the root point is this user file here. We need to find the other networks and pivot to them.

&#x20;

Did a ping sweep, and our IP here is 172.19.0.2, and there's only one other host.

&#x20;

!\[\[05\_GoodGames\_image0023.png]]

Safe to say that this is the other host. We could try an SSH in from here, as I bet this thing has SSH, otherwise there's no other form of access.

&#x20;

Safest thing to do is just SSH in to the user Augustus on that machine from this docker container.

&#x20;

!\[\[05\_GoodGames\_image0024.png]]

Cool!

&#x20;

Inside the /var/www file, I took a look at the auth.py and saw database credentials.

&#x20;

!\[\[05\_GoodGames\_image0025.png]]

&#x20;

!\[\[05\_GoodGames\_image0026.png]]

&#x20;

This just gives us the hash for the administrator account.

I went back down to the other shell, and noticed that the linpeas.sh I downloaded into /home/augustus was created as a root folder.

&#x20;

!\[\[05\_GoodGames\_image0027.png]]

Gave up at this point a looked up a guide.

&#x20;

Turns out, all we had to do was copy files over to the user directory as augustus. The root shell in the container could easily make it executable by all. Then from there we

!\[\[05\_GoodGames\_image0028.png]]

When we create this file, it is executable by root. However, if we switch back down to the container root user, we can vary this and make it executable by all.

&#x20;

!\[\[05\_GoodGames\_image0029.png]]

&#x20;

!\[\[05\_GoodGames\_image0030.png]]

From there, we can grab the root flag.

&#x20;

&#x20;
