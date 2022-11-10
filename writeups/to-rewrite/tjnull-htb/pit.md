# Pit

Pit

Wednesday, 2 March 2022

11:01 pm

This is a Linux machine.

The target IP is 10.10.10.241.

My IP is 10.10.16.9.

&#x20;

Without further ado, enumeration time.

!\[\[65\_Pit \_image001.png]]

There's a firewall...hmm..

Let's try a stealthier scan, which does not really show anything else.

&#x20;

So we make do with what we have.

Visiting the web page reveals this is a Red Hat machine.

!\[\[65\_Pit \_image002.png]]

Ran a directory scan on this website, really suspicious.

&#x20;

Seems to be running on .html, which indicates that we don't need to bother with php and jsp or whatever.

Wanted to do some dns recon on this website as well, just in case...Didn't really find much yet.

&#x20;

Let's visit the port 9090, which reveals itself to be a HTTPS server.

!\[\[65\_Pit \_image003.png]]

It has a log in, but let's check out that certificate first.

&#x20;

!\[\[65\_Pit \_image004.png]]

Add this to our hosts file.

&#x20;

!\[\[65\_Pit \_image005.png]]

Well.

Anyways, when viewing the page source, we can see that there is some obfuscated code.

&#x20;

!\[\[65\_Pit \_image006.png]]

Does not seem to be of use though.

&#x20;

At this point, I tried to nmap some UDP ports in case I missed something. Could be something of use there.

Interestingly, there's one port of use here.

No doubt in my mind that there are a lot of UDP ports, hence I just use the top 15 ports.

SNMP is enabled on the server! And this is SNMPv1, even better.

!\[\[65\_Pit \_image007.png]]

Perhaps that is the public string there.

&#x20;

Taking a look at hacktricks, we can see it is possible for this to lead to an eventual RCE.

!\[\[65\_Pit \_image008.png]]

This seemed to work and produces a long ass output. We can confirm that the hostname is indeed pit.htb.

&#x20;

From here, there was nothing that was notable (or things that I knew).

I tried a few commands from hacktricks.

!\[\[65\_Pit \_image009.png]]

&#x20;

!\[\[65\_Pit \_image0010.png]]

&#x20;

!\[\[65\_Pit \_image0011.png]]

&#x20;

This gave me some form of stuff to work with.

Did the same command outputted it to a file just to see what else I could do, this time wanting to enumerate everything.

&#x20;

!\[\[65\_Pit \_image0012.png]]

When looking through the file, I came across this:

!\[\[65\_Pit \_image0013.png]]

Seeddms?

This was within the var/www/html file, so let's take a look at that directory over there.

This let me in!

&#x20;

!\[\[65\_Pit \_image0014.png]]

&#x20;

I know the user is michelle, and I don't know the password.

I tried a few times.

Surprisingly, the password is michelle as well.

!\[\[65\_Pit \_image0015.png]]

There's a CHANGELOG there, and it is true that the version of SeedDMS 5.1.10 is vulnerable to an authenticated RCE.

&#x20;

Looking in this folder, it seems that we are allowed to add folders!

!\[\[65\_Pit \_image0016.png]]

Well, PHP reverse shell?

!\[\[65\_Pit \_image0017.png]]

Didn't work, it does not let me execute it. Need to find a way to execute the code.

Looking at the exploit from previous versions, we see this.

!\[\[65\_Pit \_image0018.png]]

Okay. So from here we can try to execute our file with a document id of 29.

&#x20;

!\[\[65\_Pit \_image0019.png]]

Seems that it does not allow us.

&#x20;

Perhaps I can change the file to something else, like a command line execution so I'm at least allowed to browse files within the server.

&#x20;

!\[\[65\_Pit \_image0020.png]]

This should work nicely.

&#x20;

!\[\[65\_Pit \_image0021.png]]

&#x20;

!\[\[65\_Pit \_image0022.png]]

Right, we have some RCE now. Let's try to look around.

&#x20;

!\[\[65\_Pit \_image0023.png]]

&#x20;

!\[\[65\_Pit \_image0024.png]]

Conf files are always good.

&#x20;

!\[\[65\_Pit \_image0025.png]]

&#x20;

!\[\[65\_Pit \_image0026.png]]

It seems that the cmd.php file is deleted after a certain period of time.

&#x20;

Nothing here, let's look around more.

&#x20;

!\[\[65\_Pit \_image0027.png]]

Another conf file, let's take a look.

&#x20;

!\[\[65\_Pit \_image0028.png]]

Looks like the same file, let's check it out.

&#x20;

!\[\[65\_Pit \_image0029.png]]

Ooo.

&#x20;

SSH does not work with this.

&#x20;

But the login on port 9090 works!

!\[\[65\_Pit \_image0030.png]]

There's an actual terminal for this...

!\[\[65\_Pit \_image0031.png]]

Grab that user flag.

From here, reverse shell into my terminal.

&#x20;

!\[\[65\_Pit \_image0032.png]]

&#x20;

!\[\[65\_Pit \_image0033.png]]

This is a funny shell.

!\[\[65\_Pit \_image0034.png]]

Look at the background.

!\[\[65\_Pit \_image0035.png]]

Eventually...this gave me a stable enough shell.

&#x20;

Immediately, we should notice that this is not our usual Linux CLI, guess it's a Red Hat kind of thing.

Took a break, came back, had trouble reconnecting to the port 9090 server, troubleshooted and reset a few times wondering what's wrong.

Listing processes here was no use, and no script could be transferred here, hence I relied on using SNMP again to enumerate out the processes which it did so effectively previously.

&#x20;

Looking back at the processes listed, I found a few.

!\[\[65\_Pit \_image0036.png]]

&#x20;

!\[\[65\_Pit \_image0037.png]]

&#x20;

!\[\[65\_Pit \_image0038.png]]

Checked the first one, which was monitor, and found it to be quite peculiar.

&#x20;

{width="5.09375in" height="1.8645833333333333in"}

Seems that for every script within that directory, it would execute the script fully.

&#x20;

!\[\[65\_Pit \_image0040.png]]

The monitoring script there cannot be written into or viewed.

However, I was permitted to create files.

{width="5.28125in" height="1.15625in"}

Let's do the simple ping test to check whether this is executed as root or something.

This took a long time...and I realised that this might not be a crontab, and began to wonder if we would force the execution.

&#x20;

For some reason, this does not execute when it comes to pings, however I was confident that this was the way to go when trying this.

&#x20;

As such, I just changed the file to instead echo my public key into the authorised keys folder of root.

Didn't work, so let's try appending other keys into the authorized\_keys folder through creating our own pair of keys.

Then I realised that the script has to be name check...

Re-did it.

{width="8.96875in" height="2.4479166666666665in"}

&#x20;

Afterwards, I just waited and sooner or later, could SSH in.

!\[\[65\_Pit \_image0043.png]]

Through the walkthroughs, I learnt that I could have sped up this process by mainly using the snmpwalk to force an execution.

I tested this.

&#x20;

Interestingly, this does not work out as per planned. So I tried with a different script, just by using touch to create a file within the root directory.

&#x20;

I could not get it to work with pings or anything. I don't know how these people figured that one out but still, rooted!

&#x20;

&#x20;
