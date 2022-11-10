# Trick

Trick

Tuesday, June 21, 2022

2:25 PM

Nmap scan:

!\[\[27\_Trick\_image001.png]]

&#x20;

!\[\[27\_Trick\_image002.png]]

&#x20;

Port 80:

!\[\[27\_Trick\_image003.png]]

&#x20;

When looking at the page source, we can see how there is some form of SB Forms being used.

!\[\[27\_Trick\_image004.png]]

&#x20;

There's an API token to get and perhaps there are some exploits.

&#x20;

Inputting email:

!\[\[27\_Trick\_image005.png]]

&#x20;

So this form is not being activated and there is something to do with an API token being needed.

&#x20;

Burpsuite requests:

!\[\[27\_Trick\_image006.png]]

&#x20;

The version usd here is sb-forms 0.4.1. We can also observe that these requests make use of HTTP/2.

!\[\[27\_Trick\_image007.png]]

&#x20;

HTTP Smuggling is one possibility to get out the code that is being used, since posts requests are not made over HTTP but over JS.

&#x20;

Gobuster and nikto reveal nothing of interest.

&#x20;

Port 25:

{width="5.25in" height="1.4895833333333333in"}

&#x20;

Nmap was wrong and we can indeed connect to this machine.

So it appears this is a Debian based system that is run on the end there.

&#x20;

We can try to enumerate some users for this using smtp-user-enum.

Nothing here yet.

&#x20;

Port 53:

!\[\[27\_Trick\_image009.png]]

&#x20;

We can find one subdomain:

!\[\[27\_Trick\_image0010.png]]

&#x20;

So we can find one subdomain here. Root.trick.htb, which would bring us back to the same website as before, but we can gobuster this new domain.

&#x20;

Gobuster reveals nothing yet again, but that's alright.

&#x20;

Username Enumeration:

We should find some form of users within this machine.

I used MSF because the server was laggy and MSF was the most stable so...

&#x20;

User Enumeration:

!\[\[27\_Trick\_image0011.png]]

&#x20;

This took a while but eventually found nothing much.

&#x20;

The other option now is to look at DNS, which reveals something useful.

{width="7.8125in" height="2.5625in"}

&#x20;

There's this prepod-payroll thingy here.

&#x20;

We can add that to our hosts file and visit the website.

We can see that it's a different page altogether.

&#x20;

!\[\[27\_Trick\_image0013.png]]

&#x20;

Page Source:

!\[\[27\_Trick\_image0014.png]]

&#x20;

When searching for exploits:

{width="9.604166666666666in" height="2.1875in"}

&#x20;

There are quite a few.

We can login with credentials using basic SQL Injection of ' or '1'='1 for both username and password.

!\[\[27\_Trick\_image0016.png]]

&#x20;

Enumeration:

From the Users tab, we can find that the username of the administrator is this:

!\[\[27\_Trick\_image0017.png]]

&#x20;

When checking the edit user tab, we can find this:

!\[\[27\_Trick\_image0018.png]]

&#x20;

!\[\[27\_Trick\_image0019.png]]

&#x20;

Now we have a username and credentials.

The credentials are useless though.

&#x20;

LFI:

When testing the page parameter from index.php, we can find that it is vulnerable to LFI.

!\[\[27\_Trick\_image0020.png]]

&#x20;

We can get the index.php page and view the PHP code behind it.

We can decode this and then do some code review for it.

&#x20;

Vulnerable function:

!\[\[27\_Trick\_image0021.png]]

&#x20;

There seems to be an automatic appending of the .php extension.

&#x20;

So let's review the facts:

* We have SQL Injection vulnerability within the login page that can be used to login and also write shells.
* We have an LFI that can be used to read files
* This web page in general is quite insecure with the way it handles data, and I'm sure there's some secret functions somewhere that's just not on this page.

&#x20;

With all of this information, it's likely that the secret function is somewhere on this page. Just need to find it.

When creating a user, we can see there is a POST request made to this page.

!\[\[27\_Trick\_image0022.png]]

&#x20;

Ajax.php with an action it seems.

When testing this, we can change it to something else like save\_payroll.

!\[\[27\_Trick\_image0023.png]]

&#x20;

It would reveal the web directory and hence we can now write shells into the webroot.

This could work, if it wasn't a blind SQL Injection. I'm inclined not to try this method as it's long and very painful.

&#x20;

Plus SQLMap is not allowed on OSCP...

&#x20;

Let's think about other stuff.

So we already know there's a /var/www/payroll, for the domain.

There might probably be other domains or something within the machine.

This machine has already shown to be tricky with the subdomains, hence I believe there's another one in hiding.

&#x20;

!\[\[27\_Trick\_image0024.png]]

&#x20;

Spot on.

&#x20;

Marketing:

!\[\[27\_Trick\_image0025.png]]

&#x20;

!\[\[27\_Trick\_image0026.png]]

&#x20;

Another page parameter, we can test for LFI.

!\[\[27\_Trick\_image0027.png]]

&#x20;

Works, and we can see the username is michael.

&#x20;

We can check for SSH keys, and we do find one:

!\[\[27\_Trick\_image0028.png]]

&#x20;

We can then take this key and try to re-format it to make it work with SSH.

&#x20;

SSH in as michael:

{width="5.5in" height="1.96875in"}

&#x20;

Flag:

!\[\[27\_Trick\_image0030.png]]

&#x20;

PE:

We can find that michael is part of the security group.

!\[\[27\_Trick\_image0031.png]]

&#x20;

Michael can also restart fail2ban.

!\[\[27\_Trick\_image0032.png]]

&#x20;

This was something interesting.

&#x20;

I still ran LinPEAS to check on everything else for me.

&#x20;

!\[\[27\_Trick\_image0033.png]]

&#x20;

We can write to this file.

!\[\[27\_Trick\_image0034.png]]

&#x20;

Basically, we can create new conf files and stuff for fail2ban.

In this case, we can abuse the iptables-multiport.conf.

!\[\[27\_Trick\_image0035.png]]

&#x20;

{width="5.583333333333333in" height="0.5in"}

&#x20;

Then we can restart the service and begin the brute forcing of SSH.

!\[\[27\_Trick\_image0037.png]]

&#x20;

This didn't really work out well, so we can change it such that we are to get something else..

Tried this instead.

!\[\[27\_Trick\_image0038.png]]

&#x20;

Then brute force again.

Again, didn't work.

&#x20;

In this case, I started using a bash script found here.

[https://github.com/rvizx/fail2ban/blob/main/fail2ban](https://github.com/rvizx/fail2ban/blob/main/fail2ban)

&#x20;

This was a script that would automatically make stuff for me.

!\[\[27\_Trick\_image0039.png]]

&#x20;

Then we just have to wait a bit I guess.

Eventually, we get a root shell.

{width="6.645833333333333in" height="0.8229166666666666in"}

&#x20;

Flag:

!\[\[27\_Trick\_image0041.png]]

&#x20;

Pwned!

That was a useful script found on github.

&#x20;

&#x20;

&#x20;
