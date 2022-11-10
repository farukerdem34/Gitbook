# Falafel

Falafel

Thursday, 3 March 2022

10:50 pm

This is an **Insane** Linux machine.

My IP is 10.10.16.9. and 10.10.16.3.

The target IP is 10.10.10.73.

&#x20;

Enumeration!

A quick search finds out that HTTPS and SSH are on the machine.

&#x20;

!\[\[66\_Falafel\_image001.png]]

&#x20;

When enumerating into specifics:

!\[\[66\_Falafel\_image002.png]]

&#x20;

Website:

!\[\[66\_Falafel\_image003.png]]

&#x20;

Anyways, it was a simple website.

Ran a directory scan as well, just to check.

Seems to be running using PHP, and there are a few interesting folders located within this.

&#x20;

!\[\[66\_Falafel\_image004.png]]

&#x20;

Robots.txt:

!\[\[66\_Falafel\_image005.png]]

Cyberlaw.txt

!\[\[66\_Falafel\_image006.png]]

&#x20;

Seems like chris is the user, and he reports some image upload stuff. Fitting, seeing as to how there is an /uploads folder, presumably where I will get my shell from.

There are a few more emails as well. Cyber protection is on the login form, and this tells me that there is something to do with

&#x20;

Anyways, there's this login page.

&#x20;

!\[\[66\_Falafel\_image007.png]]

Simple enough.

When trying username as admin, the error was:

!\[\[66\_Falafel\_image008.png]]

&#x20;

When trying any other user, it told me this:

!\[\[66\_Falafel\_image009.png]]

&#x20;

Seems that we can use this to enumerate out users.

!\[\[66\_Falafel\_image0010.png]]

We know chris is someone on this server. I suspected this could be the use of a Boolean condition that we can use to determine whether statements are true or not. This could be used to perform blindSQL injections for our web server.

&#x20;

Anyways, I intercepted the response and ran an SQLMap in case.

!\[\[66\_Falafel\_image0011.png]]

My response.

&#x20;

I wanted to see if they allowed brute forcing using hydra.

Seems that brute forcing does not get us our way.

&#x20;

I tried initially with SQLMap. I knew that this had a boolean condition there, so I wanted to test some blind SQLi.

Googled around on how to draw databases from this.

Turns out we just need to append the --level 5 and risk=3 condition there. I found how to append boolean conditions on SQLMap.

&#x20;

The true string would be the "Wrong identification : chris" part. I know that we can simply just use the --string parameter to append this condition, making it our boolean condition.

&#x20;

Left it there to run while I grabbed some water.

{width="10.635416666666666in" height="0.5104166666666666in"}

Ooo.

&#x20;

!\[\[66\_Falafel\_image0013.png]]

&#x20;

Great!

!\[\[66\_Falafel\_image0014.png]]

I dumped the entire database from this.

&#x20;

{width="8.375in" height="2.1354166666666665in"}

Interesting! Now we know the password for chris.

&#x20;

!\[\[66\_Falafel\_image0016.png]]

Cool! With this, we can now view all of the other php files that were present but had file sizes of 0 or redirected me to other places.

Juggling...This website was PHP...I googled and it referred me to PHP Type Juggling?

&#x20;

There is a clear hint because he compares juggling and PHP, really clear.

PHP Type Juggling has something to do with how PHP compares the values even when they have different types.

&#x20;

This makes use of the == operand, and its loose comparisons.

It arises when the == operand is employed instead of the === operand, which would allow for the application returning unintended answers to the true or false statement.

&#x20;

I have a feeling that being able to return true statements would allow us to login as the admin!

Now, there's a second part to this called magic hashes.

&#x20;

{width="10.989583333333334in" height="5.364583333333333in"}

&#x20;

I just need to get some kind of hash to create this basically. I need to create a number that has a similar hash to the hash I have found from the SQLMap injection.

&#x20;

Conveniently, [this](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Type%20Juggling/README.md) gave us a table full of hashes that we can try.

&#x20;

!\[\[66\_Falafel\_image0018.png]]

Tried the first one and this worked! We can login as the administrator using the first string suggested.

&#x20;

!\[\[66\_Falafel\_image0019.png]]

Now, it's just a simple upload reverse shell kind of thing.

Created the usual script.

!\[\[66\_Falafel\_image0020.png]]

Let's try to upload this and access it somewhere.

We need to use a python server to upload this.

&#x20;

!\[\[66\_Falafel\_image0021.png]]

This does not work because it checks for extensions...hmm...

Based on the cyberlaw.txt, I suppose this only accepts image files.

&#x20;

!\[\[66\_Falafel\_image0022.png]]

Tried it out with some other command, and this worked out. Now, let's try to access this from the /uploads folder.

&#x20;

!\[\[66\_Falafel\_image0023.png]]

&#x20;

&#x20;

!\[\[66\_Falafel\_image0024.png]]

Also works.

But this does not get me my shell.

&#x20;

I then tried the next trick, which was to upload a file with an absurdly long name.

&#x20;

!\[\[66\_Falafel\_image0025.png]]

Right, so to test I put in a very long string instead.

Afterwards, let's take the upload request from Burpsuite.

&#x20;

!\[\[66\_Falafel\_image0026.png]]

&#x20;

This didn't work out favourably because well, there's a lot of stuff altogether.

Took a while to make it work, but gave up after a while until I saw this.

&#x20;

!\[\[66\_Falafel\_image0027.png]]

I was on the right track but could not make it work, so I continued to try.

I gave up on curl and switched to using Burpsuite.

&#x20;

Found this.

&#x20;

!\[\[66\_Falafel\_image0028.png]]

&#x20;

Too long?

Tried with 100 less "A"s, which worked.

&#x20;

Tried with 250 and it was too long.

&#x20;

!\[\[66\_Falafel\_image0029.png]]

&#x20;

Tried with 228, and it worked.

&#x20;

All in all, it seems that the total chracters is set at a max of 236. Anything more would cause a truncation in the name.

&#x20;

!\[\[66\_Falafel\_image0030.png]]

Also, it seems that it is shortening my codes. This would mean that I am able to effectively eliminate the .jpg extension when uploading to the machine and still get it to upload.

!\[\[66\_Falafel\_image0031.png]]

Got it there!

So from here, we need to upload the payload into the web server. This would appear as a .php file inside the server, and we should be able to execute it directly.

&#x20;

So, the VPN IP changed to 10.10.16.3 from now. Little bit of confusion there as to why my web server was not receiving anything.

Once uploaded, we can test it out.

!\[\[66\_Falafel\_image0032.png]]

&#x20;

Clearly, this works and we have RCE!

Load in a reverse shell there.

&#x20;

!\[\[66\_Falafel\_image0033.png]]

&#x20;

!\[\[66\_Falafel\_image0034.png]]

Foothold at last.

&#x20;

There are 2 users in this machine.

!\[\[66\_Falafel\_image0035.png]]

Upon reviewing the ports that are open, we can see that MySQL is open on the server.

&#x20;

!\[\[66\_Falafel\_image0036.png]]

Good to know, and I went searching for some form of credentials.

&#x20;

Inside the /var/www/html folder, there's this bit here.

&#x20;

{width="6.8125in" height="4.59375in"}

Interesting. From here, we can see why there are vulnerabilities due to the use of the == operand in the if statement for passwords. This whole thing allowed for the magic hashes from just now.

I remembered seeing a connection.php within the directories just now, but it had a size of 0. I wanted to see what it contained here.

&#x20;

!\[\[66\_Falafel\_image0038.png]]

Alright, some credentials.

&#x20;

Tried to su, and this worked!

!\[\[66\_Falafel\_image0039.png]]

Grab the user flag, and I switched to SSH for stability.

&#x20;

We need to find a way to get into the other user, and then into root. Seems that the other user's home directory is not accessible to us, and I bet that there are certain things there.

&#x20;

Moshe cannot run sudo, so that's out of the question.

Ran a linpeas just to check, and saw this.a

Both of the users are under a lot of groups as well.

!\[\[66\_Falafel\_image0040.png]]

&#x20;

Okay, I wanted to see what users are on this machine now.

!\[\[66\_Falafel\_image0041.png]]

&#x20;

Seems I was right, and yossi is logged in but just there.

Furthermore, I can see that Yossi's password was changed here.

{width="10.729166666666666in" height="0.7916666666666666in"}

&#x20;

Both of the users seem to be part of video and disk groups, and I wanted to check them out.

!\[\[66\_Falafel\_image0043.png]]

&#x20;

!\[\[66\_Falafel\_image0044.png]]

Something to do with that fb0 file, so let's try it out.

For now, I just transferred it to my machine as a fb0.raw file, just to see what was there.

&#x20;

Now, I wanted to see if I could open it. It says to use Gimp but didn't exactly work for me.

Let's grab the screen resolution first.

&#x20;

!\[\[66\_Falafel\_image0045.png]]

Ok, this is the width and the height.

&#x20;

!\[\[66\_Falafel\_image0046.png]]

This is definitely an image now. Just need to find some place to open it somehow.

!\[\[66\_Falafel\_image0047.png]]

Open it in Gimp.

We will get this weird looking picture.

!\[\[66\_Falafel\_image0048.png]]

We can change the height and width to what we found earlier.

!\[\[66\_Falafel\_image0049.png]]

Can sort of make out the passwd word.

Changed the image type and this was clear.

!\[\[66\_Falafel\_image0050.png]]

That's a password!

&#x20;

Save that somewhere, and now we can try to su into moshe.

!\[\[66\_Falafel\_image0051.png]]

Alright.

&#x20;

Now, we are in yossi, which is part of another group called the disk and cdrom group. I suspected that there might be something mounted on the machine, so I took a look.

&#x20;

!\[\[66\_Falafel\_image0052.png]]

Hmm.

&#x20;

We can access this drive using debugfs and view the root file actually.

{width="4.125in" height="2.2604166666666665in"}

From here we can actually get the flag, but I want to SSH into root to make sure I really privilege escalate this machine.

Let's grab the private key.

!\[\[66\_Falafel\_image0054.png]]

SSH in and we are root.

&#x20;

!\[\[66\_Falafel\_image0055.png]]

That was one hell of a machine, really great privilege escalation choices that were put here.

&#x20;

Always check everything!

&#x20;

&#x20;
