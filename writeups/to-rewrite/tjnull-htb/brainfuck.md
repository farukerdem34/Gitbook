# BrainFuck

**BrainFuck**

Friday, 25 February 2022

9:19 pm

This is an **Insane** Linux machine.

The target IP is 10.10.10.17.

My IP is 10.10.16.9.

&#x20;

Enumeration time!

A quick Nmap reveals this.

!\[\[58\_BrainFuck\_image001.png]]

While doing a detailed scan, let's review the https website.

&#x20;

Seems to be a default nginx website.

!\[\[58\_BrainFuck\_image002.png]]

Upon inspection of the certificate, this is what we get.

&#x20;

!\[\[58\_BrainFuck\_image003.png]]

&#x20;

Take note of that Email, as there are SMTP, POP3 and IMAP ports that are open. Connection to the SMTP port to VRFY this user works.

!\[\[58\_BrainFuck\_image004.png]]

&#x20;

At this point, the Nmap scans came back.

!\[\[58\_BrainFuck\_image005.png]]

Dovecot...Hmm.

I did a directory scan on the HTTPS website and moved on to the email ports.

I first did an Nmap scan to determine what commands could be executed on this host.

&#x20;

Directory scan came back negative. Hmm...

!\[\[58\_BrainFuck\_image006.png]]

&#x20;

These are the commands that are available, again nothing much.

&#x20;

So we are left with just this.

&#x20;

Let's try other methods. I first tried accessing the website with the [https://brainfuck.htb](https://brainfuck.htb) after adding it to hosts.

&#x20;

This gave me something new.

!\[\[58\_BrainFuck\_image007.png]]

Wordpress. There's the same email as before.

As usual, there's a login present on the page.

&#x20;

!\[\[58\_BrainFuck\_image008.png]]

Let's do our usual Wpscan against this.

!\[\[58\_BrainFuck\_image009.png]]

Take note there's this open ticket mechanism working, and it seems to be out of date.

!\[\[58\_BrainFuck\_image0010.png]]

Great, there seem to be a few possible vulnerabilities for this.

!\[\[58\_BrainFuck\_image0011.png]]

There seems to be an invalid method of which there are cookies set in the PE one, let's take a look at some scripts that can do that for us.

&#x20;

!\[\[58\_BrainFuck\_image0012.png]]

We can create some exploit.html frames to make this happen.

&#x20;

I used the username admin and the one email I know of.

&#x20;

Create the frames.

!\[\[58\_BrainFuck\_image0013.png]]

Create a python server and visit ourselves on that server.

!\[\[58\_BrainFuck\_image0014.png]]

Interestingly, this does not work. Let's try with other usernames like administrator. Also does not work. At this point I was sure that I was missing something.

&#x20;

I viewed the certificate again for brainfuck.htb, and this returned two more DNS servers.

&#x20;

!\[\[58\_BrainFuck\_image0015.png]]

&#x20;

Anways, I tried stripping the URL to just brainfuck.htb, because this was a cookie thing, so perhaps this PHP site does not exist on the website. I was right!

!\[\[58\_BrainFuck\_image0016.png]]

&#x20;

Now, let's look around, at some stuff that we can find on this website.

Let's see what we can do on this admin account. I wanted to check the profile of which I was logged in as, which was just admin.

&#x20;

I saw there was this plugins tab here.

!\[\[58\_BrainFuck\_image0017.png]]

And there's this SMTP thingy.

&#x20;

There's a password written on the website.

!\[\[58\_BrainFuck\_image0018.png]]

This password is just here, we can just view the page element to see what it is.

&#x20;

!\[\[58\_BrainFuck\_image0019.png]]

Got the password!

&#x20;

Tried all 3 ports, only one seemed to let me log in, and that was port 110.

&#x20;

!\[\[58\_BrainFuck\_image0020.png]]

Cool.

&#x20;

So there were 2 messages.

!\[\[58\_BrainFuck\_image0021.png]]

I saw both, and the second one is better.

!\[\[58\_BrainFuck\_image0022.png]]

There's more credentials, and this is pointing to a secret forum. There's a secret looking website I got before this. Let's go there.

!\[\[58\_BrainFuck\_image0023.png]]

Another forum page to log into.

There are then 3 posts displayed to us.

&#x20;

!\[\[58\_BrainFuck\_image0024.png]]

&#x20;

!\[\[58\_BrainFuck\_image0025.png]]

Ok...Then there's this.

{width="7.145833333333333in" height="6.833333333333333in"}

Now this, is encryption. I can tell there's another website, with some form of text based encryption. I guessed around, and identified that it might be Vignere Cipher.

&#x20;

!\[\[58\_BrainFuck\_image0027.png]]

Cool key.

&#x20;

Anyways, now we can get that website.

&#x20;

Got it.

&#x20;

!\[\[58\_BrainFuck\_image0028.png]]

Let's head there.

And that gives us the user's id\_Rsa file. Of course, it's password protected.

The trick is that the user is able to brute force it.

&#x20;

Time to bust out SSH2john.

!\[\[58\_BrainFuck\_image0029.png]]

Then brute force it using john.

{width="9.916666666666666in" height="2.8541666666666665in"}

Now openssl decrypt it.

!\[\[58\_BrainFuck\_image0031.png]]

Now we can try to SSH to orestis using this key.

!\[\[58\_BrainFuck\_image0032.png]]

We're in.

Grab the user flag and we continue from here.

&#x20;

There are a few things within this directory.

!\[\[58\_BrainFuck\_image0033.png]]

Hmm.

&#x20;

First of all, there's this weird encrypt.sage thing here.

!\[\[58\_BrainFuck\_image0034.png]]

It's actually a code for the root.txt.

&#x20;

So it looks like it takes it, and opens the output.txt, which has an encrypted password, converts it to hex, and does RSA with it basically.

Debug.txt has the RSA numbers, p, q and e.

&#x20;

The max bits this goes to is 1024, and e is also a random number from phi. This would mean that the value of e is low.

&#x20;

When running a linpeas, we are part of the lxd group here. This allows us to run lxd and mount onto a container as the root user.

&#x20;

!\[\[58\_BrainFuck\_image0035.png]]

&#x20;

This means we can do the mounting thingy again.

Basically, we can do lxc, which allows us to create custom images in order to import. We can set the privileges of this to be whatever want because we are part of this group.

&#x20;

Hence, when we start this, we would mount it onto the target computer, and then launch /bin/sh to be the root user.

&#x20;

&#x20;

!\[\[58\_BrainFuck\_image0036.png]]

Pwned!

&#x20;

&#x20;
