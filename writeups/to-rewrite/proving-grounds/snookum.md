# Snookum

Snookum

Wednesday, 16 March 2022

7:39 pm

The target IP is 192.168.159.58.

My IP is 192.168.49.159.

&#x20;

Enumeration time.

!\[\[09\_Snookum\_image001.png]]

&#x20;

!\[\[09\_Snookum\_image002.png]]

FTP server had nothing interesting.

&#x20;

Website:

!\[\[09\_Snookum\_image003.png]]

&#x20;

Looking online, I found that there is an RFI from this website.

!\[\[09\_Snookum\_image004.png]]

&#x20;

{width="3.2604166666666665in" height="0.8541666666666666in"}

&#x20;

!\[\[09\_Snookum\_image006.png]]

&#x20;

!\[\[09\_Snookum\_image007.png]]

Right, so let's get a shell there.

!\[\[09\_Snookum\_image008.png]]

Grab a rev.php from pentestmonkey.

!\[\[09\_Snookum\_image009.png]]

Use Curl to upload it, and on the listener port, we will catch a shell as apache.

!\[\[09\_Snookum\_image0010.png]]

&#x20;

There's one other user called michael on this machine.

!\[\[09\_Snookum\_image0011.png]]

Within the /var/www directory, there's a db.php file.

!\[\[09\_Snookum\_image0012.png]]

Seems like we have our password.

&#x20;

We can access the database from this:

!\[\[09\_Snookum\_image0013.png]]

&#x20;

!\[\[09\_Snookum\_image0014.png]]

Grab that password.

&#x20;

Decode it.

!\[\[09\_Snookum\_image0015.png]]

&#x20;

{width="10.90625in" height="2.71875in"}

Switch to an SSH shell.

&#x20;

{width="8.4375in" height="2.3229166666666665in"}

Ran a linpeas on this:

&#x20;

!\[\[09\_Snookum\_image0018.png]]

&#x20;

!\[\[09\_Snookum\_image0019.png]]

&#x20;

!\[\[09\_Snookum\_image0020.png]]

Welp, that's pretty obvious, we can just either change michael to have better privileges, or just add a user that has the same privilege as root.

&#x20;

{width="7.4375in" height="0.9166666666666666in"}

&#x20;

{width="10.822916666666666in" height="2.78125in"}

&#x20;
