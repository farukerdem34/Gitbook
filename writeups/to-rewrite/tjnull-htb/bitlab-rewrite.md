# Bitlab

Bitlab

Monday, 7 March 2022

11:37 pm

This is a Linux machine.

The target IP is 10.10.10.114.

My IP is 10.10.16.7.

&#x20;

Enumeration time.

Quick all port scan reveals that this contains only SSH and HTTP ports.

!\[\[71\_Bitlab\_image001.png]]

&#x20;

!\[\[71\_Bitlab\_image002.png]]

&#x20;

Website:

!\[\[71\_Bitlab\_image003.png]]

Let's try to sign in using something. Default credentials of admin:admin do not work.

&#x20;

At the footer of the page, there are these:

!\[\[71\_Bitlab\_image004.png]]

&#x20;

Exploire brings me to the projects page and stuff, where there's nothing remarkable.

&#x20;

When looking at the page source, there's this interesting little snippet here.

!\[\[71\_Bitlab\_image005.png]]

Localhost:3000?

There's also the version of gon.api.

&#x20;

!\[\[71\_Bitlab\_image006.png]]

This whole snippet of code us running as a script on the web page, and it gives hints like /help/shortcuts and other things like the localhost.

&#x20;

!\[\[71\_Bitlab\_image007.png]]

This done not really give us anything. I looked at all the URLs but the last one proved interesting.

When coped, there's this weird JS URL being executed. The rest of the URLs lead to proper pages.

&#x20;

{width="13.8125in" height="0.7708333333333334in"}

&#x20;

When beautified online, this is what we get.

Removed all the \x and changed the hex to strings, and this is what we get.

&#x20;

!\[\[71\_Bitlab\_image009.png]]

Seems like we can log in now.

When we log in, we can see this:

!\[\[71\_Bitlab\_image0010.png]]

When looking at the project named 'profile', I can see that we can edit it. Let's try our usual simple PHP shell into the machine.

&#x20;

!\[\[71\_Bitlab\_image0011.png]]

Afterwards, just merge and approve everything that can be done.

&#x20;

Now to test it, let's try this:

&#x20;

!\[\[71\_Bitlab\_image0012.png]]

&#x20;

{width="9.791666666666666in" height="2.1041666666666665in"}

Cool!

&#x20;

Reverse shell time.

&#x20;

!\[\[71\_Bitlab\_image0014.png]]

&#x20;

!\[\[71\_Bitlab\_image0015.png]]

We are in as www-data as usual.

&#x20;

Doing an ifconfig reveals that there are containers running on this machine.

&#x20;

!\[\[71\_Bitlab\_image0016.png]]

&#x20;

!\[\[71\_Bitlab\_image0017.png]]

There are a total of 5 other containers running on this.

&#x20;

I ran a linpeas to check on stuff, and what are stuff that are running on this machine.

&#x20;

!\[\[71\_Bitlab\_image0018.png]]

I ran an Nmap against 172.19.0.0/16, seeing that there is Nmap installed on this machine.

!\[\[71\_Bitlab\_image0019.png]]

It seems that these hosts are up.

&#x20;

The last one has another HTTP port that is open. Perhaps we can tunnel to one of these ports.

&#x20;

I noticed that within the web page, there was this snippets section which offered this bit of code.

&#x20;

!\[\[71\_Bitlab\_image0020.png]]

Perhaps we can use this somehow.

Look around for a while, but my lack of PHP knowledge made me refer to a walkthrough for this snippet.

{width="10.010416666666666in" height="3.125in"}

From here, we can get a password.

&#x20;

This password was the actual password of clave, and there's no need to decode it whatsoever.

!\[\[71\_Bitlab\_image0022.png]]

Grab the user flag and let's continue on.

Within the same directory, notice there's this RemoteConnection.exe, which is very interesting indeed.

&#x20;

Let's take a look at this and see what we can get from it.

Transferred using some netcat.

For now, pausing because RE takes a lot of energy...

I have to say that this box is not realistic, more of a fun challenge that I'm tired to solve.

&#x20;

I'm sure that I need to transfer this to a Windows machine to run a proper debugger instead of within a Linux VM.

&#x20;

&#x20;

&#x20;

&#x20;
