# Paper

Paper

Thursday, 24 February 2022

11:28 pm

This is a Linux machine.

The target IP is 10.10.11.143.

My IP is 10.10.16.9.

&#x20;

!\[\[01\_Paper\_image001.png]]

&#x20;

Ok, so let's view the HTTP page.

!\[\[01\_Paper\_image002.png]]

&#x20;

Default page.

Let's view the HTTPS page, while running a directory scan on this thing.

&#x20;

Same page...

!\[\[01\_Paper\_image003.png]]

Let's check the certificate.

Nothing of interest there.

&#x20;

Anyways, we know this is CentOS.

&#x20;

!\[\[01\_Paper\_image004.png]]

There has to be more to this...

I was confident that there was some kind of subdomain somewhere.

&#x20;

I appended 'paper', 'paper.htb' to my /etc/hosts lists.

&#x20;

Enumerating with anything in front of 'paper.htb' yielded no results.

I turned to trying appending things in front of paper, like '[http://something.paper](http://something.paper)'

&#x20;

Didn't really work. I analysed the stuff in Burp, to see if I can get anything of interest. Found this though.

!\[\[01\_Paper\_image005.png]]

&#x20;

!\[\[01\_Paper\_image006.png]]

Office.paper?

This must be the thing I need.

&#x20;

Add that to the hosts file and let's view the page.

!\[\[01\_Paper\_image007.png]]

&#x20;

Some kind of forum page. From here, we can determine the engine hosting this that would give us a hint on where to begin.

&#x20;

!\[\[01\_Paper\_image006.png]]

&#x20;

!\[\[01\_Paper\_image008.png]]

Wordpress Ver 5.2.3.

&#x20;

This gives us a few exploits to work with, but I took a look around the page more first.

!\[\[01\_Paper\_image009.png]]

&#x20;

Found this comment!

&#x20;

From here, we can determine the exploit to use, which basically **appends ?=static=1** at the back of the home page. This shows us a draft post we're not supposed to see.

!\[\[01\_Paper\_image0010.png]]

&#x20;

Using this link, we can access a register page. I registered and then proceeded to view it. It is a version of rocket.chat that is used like Discord.

!\[\[01\_Paper\_image0011.png]]

&#x20;

From here, we can view the general chat.

There seems to be a particular robot that is used to do stuff, and it executes code on the machine.

!\[\[01\_Paper\_image0012.png]]

&#x20;

I started a private chat with the bot and began to see what it could do.

&#x20;

!\[\[01\_Paper\_image0013.png]]

From here, I tried a simple directory traversal.

&#x20;

!\[\[01\_Paper\_image0014.png]]

&#x20;

Works! I played around with this a lot before trying other things. I saw that within this, there was a hubot file we owned.

!\[\[01\_Paper\_image0015.png]]

&#x20;

From here, there was a simple .env file. I know this file is used for all variables for bots! Using the file command, I got the contents out.

!\[\[01\_Paper\_image0016.png]]

Cool! Now I know this user is called dwight, and I have this password. Let's try SSH.

&#x20;

{width="6.645833333333333in" height="2.1979166666666665in"}

Very cool, we are in. Grab the user flag while you're at it.

&#x20;

Then from there, we can start eumerating. I got LinPeas.sh into the machine and began to enumerate out what are the possibilities.

&#x20;

!\[\[01\_Paper\_image0018.png]]

Looks promising. I went to Github and got a script.

Just search this CVE.

&#x20;

From there, got it onto the victim machine and executed.

!\[\[01\_Paper\_image0019.png]]

Executed with errors, lots of them but eventually...

&#x20;

!\[\[01\_Paper\_image0020.png]]

Pwned.
