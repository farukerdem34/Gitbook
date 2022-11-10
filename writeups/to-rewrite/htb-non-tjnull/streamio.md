# StreamIO

StreamIO

Tuesday, June 21, 2022

7:06 PM

Nmap scan:

!\[\[28\_StreamIO\_image001.png]]

&#x20;

HTTPS:

!\[\[28\_StreamIO\_image002.png]]

&#x20;

!\[\[28\_StreamIO\_image003.png]]

&#x20;

So there are two domains.

&#x20;

streamIO.htb:

!\[\[28\_StreamIO\_image004.png]]

&#x20;

When viewing in burp, we can find one user here.

!\[\[28\_StreamIO\_image005.png]]

&#x20;

Interesting bit:

!\[\[28\_StreamIO\_image006.png]]

&#x20;

Not sure if this means anything.

&#x20;

Watch.streamIO.htb:

!\[\[28\_StreamIO\_image007.png]]

&#x20;

We can leave emails there, and there's an FAQ to look at.

Viewing the page source, we can find this little bit here:

!\[\[28\_StreamIO\_image008.png]]

&#x20;

There's a Login button.

!\[\[28\_StreamIO\_image009.png]]

&#x20;

This does not do anything though, but now we know there's a login on this page somewhere.

&#x20;

&#x20;

Gobuster:

We can gobust both websites.

!\[\[28\_StreamIO\_image0010.png]]

&#x20;

!\[\[28\_StreamIO\_image0011.png]]

&#x20;

/admin?

!\[\[28\_StreamIO\_image0012.png]]

&#x20;

Nice.

&#x20;

Before gobusting the watch page, we can find that it's a PHP page.

!\[\[28\_StreamIO\_image0013.png]]

&#x20;

Gobuster:

!\[\[28\_StreamIO\_image0014.png]]

&#x20;

!\[\[28\_StreamIO\_image0015.png]]

&#x20;

Search.php:

!\[\[28\_StreamIO\_image0016.png]]

&#x20;

Looks like some form of Database already.

&#x20;

Retry gobuster with PHP:

!\[\[28\_StreamIO\_image0017.png]]

&#x20;

!\[\[28\_StreamIO\_image0018.png]]

&#x20;

!\[\[28\_StreamIO\_image0019.png]]

&#x20;

This means that the web page is accessible through including another file...whatever that means.

&#x20;

Retry Gobuster:

!\[\[28\_StreamIO\_image0020.png]]

&#x20;

!\[\[28\_StreamIO\_image0021.png]]

&#x20;

There's a login.php.

!\[\[28\_StreamIO\_image0022.png]]

&#x20;

Nothing here.

&#x20;

Let's take a look at LDAP and the other forms of AD-related stuffs.

&#x20;

KerBrute:

{width="9.885416666666666in" height="2.84375in"}

&#x20;

Only martin can be found here.

&#x20;

Right, let's review the facts:

* There's an admin page that is only accessible through includes (whatever that means)
* There are login pages here and there, and I can test these for some SQL Injection.

&#x20;

SQL Injection:

!\[\[28\_StreamIO\_image0024.png]]

&#x20;

{width="9.78125in" height="1.7083333333333333in"}

> &#x20;

Seems that this is vulnerable to SQL Injection, and we can begin dumping out stuff.

!\[\[28\_StreamIO\_image0026.png]]

&#x20;

That confirms it's vulnerable to time-based SQL Injection. We can use SQLMap to try and brute force the credentials out, otherwise it would be really hard to get them out.

!\[\[28\_StreamIO\_image0027.png]]

Then we just have to wait...It would take a while.

{width="4.46875in" height="1.1875in"}

&#x20;

So there's a user's table within this database.

&#x20;

Dumping user's table:

{width="7.083333333333333in" height="0.7604166666666666in"}

&#x20;

This one takes a long ass time...

Eventually, we do get a few hashes but I relented and looked at a guide for the password. Turns out the password we were meant to find was at id=31, whereby it took about 2 hours to get to id=16.

&#x20;

Likely the fastest way was to enumerate the first and last possible ID to get something out, but whatever. Who would've known?

&#x20;

We can crack it and then use it to login to the login.php and then the /admin panel becomes accessible to us.

!\[\[28\_StreamIO\_image0030.png]]

&#x20;

Admin Panel:

When looking around the panel, we see a unique parameter being passed.

!\[\[28\_StreamIO\_image0031.png]]

&#x20;

There's this thing here, whereby it seems very...suspicious in general.

&#x20;

This does not really work out well, as we cannot do anything with it.

However, we can fuzz the rest of that parameter to see which works.

!\[\[28\_StreamIO\_image0032.png]]

&#x20;

We can test and see what other possible stuff there is that is not normal basically.

Out of all of them, we can see one that pops up early.

!\[\[28\_StreamIO\_image0033.png]]

&#x20;

**Debug...**

!\[\[28\_StreamIO\_image0034.png]]

&#x20;

Let's try the LFI again.

!\[\[28\_StreamIO\_image0035.png]]

&#x20;

Works!

&#x20;

There was one suspicious file called master.php...which I do want to look at.

&#x20;

Master.php:

We can decode it and then view it.

&#x20;

{width="5.364583333333333in" height="2.0104166666666665in"}

&#x20;

There's this last bit here that has **eval() function,** which is known for well, basically executing code and there does not seem to be any form of sanitsation of the input here.

This seems to be the vector of which we need to exploit.

&#x20;

We need to include something like index.php and then POST an include thing, which I assume would be any file that we want.

&#x20;

We can take a cmd.php file and then base64 encode it, and then POST it to here with the data.

&#x20;

Let's try it.

&#x20;

So to construct this, here are the steps:

* We need to use the Cookie to allow for such actions to occur
* Have a parameter named **include** which would be evaluated
* Afterwards, we should key in some form of PHP code that would be evauluated.

&#x20;

We can take this bit from Hacktricks.

!\[\[28\_StreamIO\_image0037.png]]

&#x20;

The Plaintext one did not work, but it did work with the base64 one!

!\[\[28\_StreamIO\_image0038.png]]

&#x20;

We finally have RCE.

&#x20;

Shell:

!\[\[28\_StreamIO\_image0039.png]]

&#x20;

!\[\[28\_StreamIO\_image0040.png]]

&#x20;

!\[\[28\_StreamIO\_image0041.png]]

&#x20;

&#x20;

{width="5.59375in" height="1.8645833333333333in"}

&#x20;

Great.

&#x20;

PE1:

As this user, we don't have much control over anything regarding the box, and we don't even have access to the user flag.

I ran WinPEAS just in case but it seems that I did not find anything of interest.

&#x20;

We did have an SQL database of which one table was left...blank.

&#x20;

Within the index.php of the streamio.htb/admin page, we can see a password:

!\[\[28\_StreamIO\_image0043.png]]

&#x20;

There is a password and a username here. However, there wasn't a way of which we could connect to mssql unless it was through SQL Injection.

Let's try port forwarding it.

!\[\[28\_StreamIO\_image0044.png]]

^^ this was not picked up earlier by the nmap scan.

&#x20;

We can use chisel in this case.

!\[\[28\_StreamIO\_image0045.png]]

&#x20;

!\[\[28\_StreamIO\_image0046.png]]

&#x20;

{width="5.875in" height="2.625in"}

&#x20;

We can now look at the stuff in the database.

{width="9.572916666666666in" height="2.5625in"}

&#x20;

And we get a hash.

!\[\[28\_StreamIO\_image0049.png]]

&#x20;

Good password.

&#x20;

Then we can evil-winrm in.

!\[\[28\_StreamIO\_image0050.png]]

&#x20;

Flag:

!\[\[28\_StreamIO\_image0051.png]]

4fa939bbca7b79852b79a8fc85bb19bf

&#x20;

PE2:

I bloodhounded.

{width="10.875in" height="3.3125in"}

&#x20;

After uploading, we can view the permissions and stuff of this user.

I didn't find much to do with this, but I did find out martin was part of the administrator's group.

!\[\[28\_StreamIO\_image0053.png]]

&#x20;

Since we have credentials, we can think about how to proceed onwards. And perhaps some processes are being run by Martin here...

&#x20;

I ran another winpeas to check on stuff.

!\[\[28\_StreamIO\_image0054.png]]

Cool, there seem to be credentials here.

&#x20;

Transferring files:

!\[\[28\_StreamIO\_image0055.png]]

&#x20;

[https://github.com/lclevy/firepwd/blob/master/firepwd.py](https://github.com/lclevy/firepwd/blob/master/firepwd.py)

{width="2.9791666666666665in" height="0.5625in"}

Key4db does not have anything, but the github repo does say try logins.json files.

&#x20;

{width="3.28125in" height="0.4895833333333333in"}

&#x20;

!\[\[28\_StreamIO\_image0058.png]]

&#x20;

We have another credential and subdomain to look at now.

!\[\[28\_StreamIO\_image0059.png]]

&#x20;

This has nothing much for me.

But since we have credentials for JDGodd, let's take a look at his bloodhound stuff.

&#x20;

So Jdgodd has control over Core Staff:

!\[\[28\_StreamIO\_image0060.png]]

&#x20;

And Core Staff can read LAPS passwords.

!\[\[28\_StreamIO\_image0061.png]]

&#x20;

This would basically mean, with these credentials, we can add ourselves to this group, and proceed to read the LAPS password of the DC and hopefully authenticate as the administrator.

&#x20;

First, we can check which password is good.

{width="6.09375in" height="0.6979166666666666in"}

&#x20;

!\[\[28\_StreamIO\_image0063.png]]

&#x20;

So the first one is the best one. Right.

&#x20;

Now, we can begin to use BloodHound commands to execute stuff remotely.

Make sure to import PowerView.ps1 as well.

&#x20;

Exploit:

!\[\[28\_StreamIO\_image0064.png]]

&#x20;

!\[\[28\_StreamIO\_image0065.png]]

&#x20;

!\[\[28\_StreamIO\_image0066.png]]

&#x20;

Cool.

Now, we can read the LAPS password of users remotely.

{width="9.84375in" height="2.0625in"}

&#x20;

!\[\[28\_StreamIO\_image0068.png]]

&#x20;

Flag:

!\[\[28\_StreamIO\_image0069.png]]

05c37986d9a174edf970636c1014eb78

&#x20;

Cool box!

&#x20;

Things I've learnt:

* SQL Injection is a pain.
* Once the credentials for this box were found (after like hours), the rest was pretty okay.
* Needed help for the fuzzing bit regarding the debug portion, good to know wfuzz got my back.
* Take notes on the firefox cached credentials because we'll probably see them again.
* Worked on my powershell regarding the remote exploitation of ACL's over an AD network.

&#x20;

&#x20;

Overall, this box was pretty okay, the hardest portion was the fuzzing and the SQL Injection.

No possible way someone did this with the usage of manual exploitation, and if they did, hats off to them.

&#x20;
