# Monitors

Monitors

Friday, 4 March 2022

5:57 pm

This is an **Insane** Linux machine.

My IP is 10.10.16.9.

The target IP is 10.10.10.238.

&#x20;

Enumeration!

A quick scan reveals this SSH and HTTP, and a more detailed scan reveals the following:

!\[\[67\_Monitors\_image001.png]]

Okay, when visiting the site, it says this.

!\[\[67\_Monitors\_image002.png]]

Guess we need to add it into the hosts file. Anyways accessing it properly reveals this is a Wordpress sight.

&#x20;

!\[\[67\_Monitors\_image003.png]]

Wordpress scan this.

!\[\[67\_Monitors\_image004.png]]

&#x20;

!\[\[67\_Monitors\_image005.png]]

&#x20;

!\[\[67\_Monitors\_image006.png]]

As usual, this would have one login page that we might need to login to later as the admin.

I did another scan to enumerate out any users.

!\[\[67\_Monitors\_image007.png]]

Aight.

&#x20;

Looking up the spritz plugin, I found there is a RFI for it.

!\[\[67\_Monitors\_image008.png]]

Right, so we should probably expect to use that /uploads folder.

&#x20;

!\[\[67\_Monitors\_image009.png]]

So we can inject reverse shells into this. Let's try the POC first.

As for the /etc/passwd, it works well.

&#x20;

!\[\[67\_Monitors\_image0010.png]]

Now, let's do the exploit. This would have to be some kind of reverse shell or something.

Tried it out with a PHP command line code, just to see what would happen.

&#x20;

!\[\[67\_Monitors\_image0011.png]]

&#x20;

&#x20;

!\[\[67\_Monitors\_image0012.png]]

No output, let's try a different command with the use of ping perhaps.

Does not work either.

&#x20;

Right, so we either need to find where this file has been uploaded or simply inject a reverse shell into it and see what happens.

Neither of them work.

&#x20;

I was thinking, then I realised I could just try to directory traversal my way to find some creds.

&#x20;

I looked into the apache2 server, seeing if there are any other sites that are enabled.

!\[\[67\_Monitors\_image0013.png]]

&#x20;

!\[\[67\_Monitors\_image0014.png]]

Interesting, let's take make a new host for that website.

!\[\[67\_Monitors\_image0015.png]]

Another log in page...

Tried this instead.

!\[\[67\_Monitors\_image0016.png]]

&#x20;

!\[\[67\_Monitors\_image0017.png]]

Here we have some credentials, but it does not work on the wordpress account, but it works with the cacti-admin log in page.

&#x20;

!\[\[67\_Monitors\_image0018.png]]

&#x20;

!\[\[67\_Monitors\_image0019.png]]

A quick searchsploit reveals this is vulnerable to some form of SQLi.

&#x20;

!\[\[67\_Monitors\_image0020.png]]

When downloading the script, this works as an RCE script as well. Executing this and we got an admin hash.

&#x20;

!\[\[67\_Monitors\_image0021.png]]

Was supposed to give me a shell but this works too.

&#x20;

So we have this password, and I suspect this is for the admin account or something.

Anyways, this gave me a shell to work with.

!\[\[67\_Monitors\_image0022.png]]

Cool!

Changed up shells and made it nicer to use.

{width="4.9375in" height="1.3125in"}

&#x20;

Was stuck here for a while, looking at sudo and some common files or looking for database files. Eventually figured out that the directory under the user, marcus was open to view.

&#x20;

!\[\[67\_Monitors\_image0024.png]]

That backup script is not supposed to be there normally.

We can go there and look at stuff actually, but we are unable to read anything, hence this is rather useless.

Before I left it, decided to just do cat backup.sh or something to view whether there were any scripts.

&#x20;

!\[\[67\_Monitors\_image0025.png]]

Well...

&#x20;

Su to marcus using that password.

There's also another note.txt within the directory, and it's a TODO list.

!\[\[67\_Monitors\_image0026.png]]

Not sure what this means as of now.

Docker image for production use?

&#x20;

Means there's something in mount perhaps.

&#x20;

!\[\[67\_Monitors\_image0027.png]]

Another interesting file, perhaps this is being written to somehow by a crontab.

I found this within the /etc/containerd file.

&#x20;

!\[\[67\_Monitors\_image0028.png]]

This is beign somehow run as root, so we need to dig further to find out.

This was the same result, a file that is executable but is not viewable by user marcus.

&#x20;

I found this however.

&#x20;

!\[\[67\_Monitors\_image0029.png]]

Port 8443, which is a tomcat engine is open, but not accessible by us.

&#x20;

Let's try some port tunneling.

!\[\[67\_Monitors\_image0030.png]]

&#x20;

!\[\[67\_Monitors\_image0031.png]]

Hmm...

&#x20;

It seems accessible but there is no directory found.

&#x20;

Decided to so some form of gobuster directory enumeration.

Found a ton of stuff really.

!\[\[67\_Monitors\_image0032.png]]

All had a size of 0 though, so I left it running. Tried the second one and it redirected me to a login page. Code 302 means redirect. Turns out all of them redirect us, except /images.

&#x20;

!\[\[67\_Monitors\_image0033.png]]

&#x20;

!\[\[67\_Monitors\_image0034.png]]

This has an exploit to it!

!\[\[67\_Monitors\_image0035.png]]

We need to download ysoserial.jar to use this exploit.

Reading the exploit, we are to create two shell files Using ysoserial within this code.

&#x20;

!\[\[67\_Monitors\_image0036.png]]

&#x20;

!\[\[67\_Monitors\_image0037.png]]

&#x20;

This exploit was not working, so I referred to [here](https://github.com/g33xter/CVE-2020-9496) to do it.

Basically this has to do with how to get us a shell into the container through the use of ysoserial in order to generate some form of RCE through base64 encoding.

Following it exactly, I got a shell.

&#x20;

!\[\[67\_Monitors\_image0038.png]]

This was not the root shell however, there was no txt file or anything to look at.

I was in a container of some sort, and needed to break out.

&#x20;

I just knew I was in some kind of docker or something...

First we would need to check the capabilities provided within the docker.

&#x20;

!\[\[67\_Monitors\_image0039.png]]

Particularly, this has the sys\_module capability.

!\[\[67\_Monitors\_image0040.png]]

So with this, we can find a way to privilege escalate.

!\[\[67\_Monitors\_image0041.png]]

So for this exploit, we need to create a new kernel module.

Let's create a quick C script.

{width="10.635416666666666in" height="4.364583333333333in"}

From here, we have to create another file called Makefile.

&#x20;

Get the username using uname -r. And change the working directory to that.

!\[\[67\_Monitors\_image0043.png]]

Get them both into the root directory of the machine and make sure to start a listener shell. We can thankfully use wget in this machine.

&#x20;

This took a long time to make it work, because Makefile was being annoying.

Turns out we can control subl (the software I use) to make it stop putting tabs and start putting spaces instead.

&#x20;

Afterwards, run the **make** command then launch the reverse shell.

!\[\[67\_Monitors\_image0044.png]]

&#x20;

!\[\[67\_Monitors\_image0045.png]]

Rooted!

&#x20;
