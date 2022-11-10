# Forward

Forward

Thursday, July 7, 2022

2:16 PM

Nmap scan:

!\[\[75\_Forward\_image001.png]]

&#x20;

Enum4linux:

!\[\[75\_Forward\_image002.png]]

&#x20;

{width="9.8125in" height="2.1041666666666665in"}

&#x20;

!\[\[75\_Forward\_image004.png]]

&#x20;

{width="9.791666666666666in" height="2.3125in"}

&#x20;

{width="8.291666666666666in" height="3.15625in"}

&#x20;

Interesting files, and we can see some cool stuff here.

Recursively download all these files.

!\[\[75\_Forward\_image007.png]]

&#x20;

Let's check the readme.

{width="9.822916666666666in" height="1.78125in"}

&#x20;

So these are keys, and we probably need to use these somehow, and googling online tells me that these are decryptable and totally exploitable.

[https://whynotsecurity.com/blog/teamviewer/](https://whynotsecurity.com/blog/teamviewer/)

&#x20;

So as of now we just know that it is possible for us to decrypt these passwords.

{width="8.3125in" height="2.1666666666666665in"}

&#x20;

Problem is...it seems too long.

&#x20;

But some of the passwords seem legit.

{width="8.125in" height="1.625in"}

&#x20;

So from here, we can compile all the possible passwords and usernames and then brute force everything together.

&#x20;

After doing all of that, we can use the wordlists and attempt to brute force SSH, SMTP and SMB.

After all of that, it does not seem to get us anything at all... However, out of all the usernames, only one seems to have legit credentials.

What was annoying was the actually crackmapexec was not working...

&#x20;

This means that our user fox actually does have valid credentials within that thing.

&#x20;

{width="9.8125in" height="2.8229166666666665in"}

&#x20;

{width="8.5in" height="4.21875in"}

&#x20;

Now we can see a .forward file and dosbox (which might be for PE) but we can take a look at that forward file.

&#x20;

{width="4.09375in" height="1.0208333333333333in"}

&#x20;

We can see that there's a procmail thing here, and that commands are piped.

We can replace this with our own shell file.

&#x20;

!\[\[75\_Forward\_image0014.png]]

&#x20;

Then we can send mail to fox.

&#x20;

!\[\[75\_Forward\_image0015.png]]

&#x20;

{width="8.510416666666666in" height="2.2291666666666665in"}

&#x20;

Flag:

!\[\[75\_Forward\_image0017.png]]

&#x20;

PE:

Linpeas:

{width="5.520833333333333in" height="1.8333333333333333in"}

&#x20;

!\[\[75\_Forward\_image0019.png]]

&#x20;

This password works with fox.

&#x20;

And we can SSH in.

&#x20;

Earlier, I saw there was a dosbox instance, of which case it is here.

!\[\[75\_Forward\_image0020.png]]

&#x20;

To exploit this thing, we would need some kind of RDP access or we can use DOSbox on our system and get into his system.

Lots of googling later, I can find this.

{width="8.25in" height="3.1979166666666665in"}

&#x20;

We need to use the -X option, which basically allows for x11 forwarding, which dosbox happens to run on.

&#x20;

We can mount the main directory, and we can edit whatever files we want.

!\[\[75\_Forward\_image0022.png]]

&#x20;

This would basically switch directories and we can mount the system as root.

!\[\[75\_Forward\_image0023.png]]

&#x20;

!\[\[75\_Forward\_image0024.png]]

&#x20;

!\[\[75\_Forward\_image0025.png]]

&#x20;

Then we can SSH to become root.

{width="8.072916666666666in" height="2.6875in"}

&#x20;

The password was 'hello'.

&#x20;

Flag:

!\[\[75\_Forward\_image0027.png]]

&#x20;

&#x20;

&#x20;
