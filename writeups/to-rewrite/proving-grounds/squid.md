# Squid

Squid

Monday, May 30, 2022

7:09 AM

Nmap scan:

!\[\[108\_Squid\_image001.png]]

&#x20;

!\[\[108\_Squid\_image002.png]]

&#x20;

Port 3128:

!\[\[108\_Squid\_image003.png]]

&#x20;

The version would be 4.14.

When using nc to check on the port, we get some Bad Request being made:

!\[\[108\_Squid\_image004.png]]

&#x20;

When using curl with the proxy, we get this interesting little comment:

!\[\[108\_Squid\_image005.png]]

&#x20;

This proxy has to be used to access the internal network.

{width="6.96875in" height="0.75in"}

&#x20;

Finding hidden port:

!\[\[108\_Squid\_image007.png]]

&#x20;

We can see that this does not give us an error but rather a hidden web server that we can access.

&#x20;

Adding proxy:

!\[\[108\_Squid\_image008.png]]

&#x20;

!\[\[108\_Squid\_image009.png]]

&#x20;

There's a phpmyadmin page, and it seems to be using SQL here.

We can try to login using some basic credentials.

!\[\[108\_Squid\_image0010.png]]

&#x20;

This lets us in.

!\[\[108\_Squid\_image0011.png]]

&#x20;

To gain a shell, we need to create a new database.

&#x20;

!\[\[108\_Squid\_image0012.png]]

&#x20;

!\[\[108\_Squid\_image0013.png]]

&#x20;

From there, we can then run some SQL Queries.

!\[\[108\_Squid\_image0014.png]]

&#x20;

We can then look for the webroot file.

!\[\[108\_Squid\_image0015.png]]

&#x20;

Which is in phpinfo() that we found earlier.

!\[\[108\_Squid\_image0016.png]]

&#x20;

Then we can inject a web shell.

!\[\[108\_Squid\_image0017.png]]

&#x20;

!\[\[108\_Squid\_image0018.png]]

&#x20;

From here, we can gain a shell.

!\[\[108\_Squid\_image0019.png]]

&#x20;

{width="4.958333333333333in" height="0.8125in"}

&#x20;

{width="8.25in" height="2.2916666666666665in"}

&#x20;

Flag:

!\[\[108\_Squid\_image0022.png]]

5dfb357fdb9bee751a79b9ea225d7535

&#x20;

However, we are not the system user yet.

&#x20;

PE:

!\[\[108\_Squid\_image0023.png]]

&#x20;

Running winpeas, we gain some output information, including that this versions is vulnerable to a few kernel exploits.

&#x20;

!\[\[108\_Squid\_image0024.png]]

&#x20;

Kernel Exploits:

!\[\[108\_Squid\_image0025.png]]

&#x20;

There are also some interesting files:

{width="10.427083333333334in" height="1.4583333333333333in"}

&#x20;

When looking at the schtasks:

!\[\[108\_Squid\_image0027.png]]

&#x20;

We can see something very interesting there.

Seems that this user can create schtasks or something...

&#x20;

We can then create an easy task like so:

!\[\[108\_Squid\_image0028.png]]

&#x20;

After waiting, we would gain a shell as the same user but with higher privileges.

{width="7.333333333333333in" height="2.2916666666666665in"}

&#x20;

!\[\[108\_Squid\_image0030.png]]

&#x20;

It seems that we are able to increase our privileges in this manner.

Afterwards, we need to create another shell to gain more privileges.

[https://itm4n.github.io/localservice-privileges/](https://itm4n.github.io/localservice-privileges/)

&#x20;

This link has some useful information regarding that. We need to specify the privileges that we want from the shell using Powershell.

&#x20;

Afterwards, to launch this task, we would need to set a Trigger time of which it would execute.

!\[\[108\_Squid\_image0031.png]]

&#x20;

We could do it this method, but we also can use something like FullPowers to do it for us.

&#x20;

{width="5.864583333333333in" height="2.3541666666666665in"}

&#x20;

!\[\[108\_Squid\_image0033.png]]

&#x20;

We notice we have SeImpersoantePrivilege now, which means we can easily printspoofer this.

!\[\[108\_Squid\_image0034.png]]

&#x20;

Flag:

!\[\[108\_Squid\_image0035.png]]

&#x20;

&#x20;
