# UC

UC404

Sunday, June 26, 2022

12:29 PM

Nmap scan:

!\[\[59\_UC404\_image001.png]]

&#x20;

!\[\[59\_UC404\_image002.png]]

&#x20;

!\[\[59\_UC404\_image003.png]]

&#x20;

I see squid, which means we may need to proxy traffic through there.

There's also this github repository that we can download.

&#x20;

When downloaded through wget recursive, we can see how it's the entire directory that is present.

&#x20;

So I found this /pages/forms/advanced.html file that had an upload.

!\[\[59\_UC404\_image004.png]]

&#x20;

Interesting that it straight up does not work, and gives a 404...

Weird.

&#x20;

We can take a look at the rest being made here.

!\[\[59\_UC404\_image005.png]]

&#x20;

Interestingly, this send a POST request to the target-url of the page, and is one of the only parts of the page that make POST requests to somewhere else within the page.

&#x20;

Well, there's nothing much left to do with code analysis here.

We can try to gobuster this website to see what else we can find.

&#x20;

Gobuster:

!\[\[59\_UC404\_image006.png]]

&#x20;

We can find this unique directory here:

!\[\[59\_UC404\_image007.png]]

&#x20;

!\[\[59\_UC404\_image008.png]]

&#x20;

This appears to be some form of PHP website, and it seems to have some forgot password thing and account creation.

!\[\[59\_UC404\_image009.png]]

&#x20;

Interestingly, when trying to login, it appears that stuff behind this page does not exist.

!\[\[59\_UC404\_image0010.png]]

&#x20;

So this means that perhaps, because all other alternatives have already been exhausted, we can try to fuzz this page somehow.

&#x20;

Forgot Password Source Code:

!\[\[59\_UC404\_image0011.png]]

&#x20;

Interesting, there seems to be some form of PHP code injection or something here, but they are blacklisting some characters.

It says that it must receive a variable from the HTML Form and then it can send a message. So I tried the %0a technique with both POST and GET requests, and was surprised.

!\[\[59\_UC404\_image0012.png]]

&#x20;

Now, we can gain a reverse shell.

&#x20;

Shell:

!\[\[59\_UC404\_image0013.png]]

&#x20;

{width="7.0in" height="1.5625in"}

&#x20;

Flag:

!\[\[59\_UC404\_image0015.png]]

440c6193fe743e7d2fa64745369fcbce

&#x20;

PE:

We can view the backups to see some stuff.

!\[\[59\_UC404\_image0016.png]]

&#x20;

{width="10.072916666666666in" height="2.6041666666666665in"}

&#x20;

And now we have access to the user brian.

{width="8.322916666666666in" height="4.3125in"}

&#x20;

Then we can find this:

!\[\[59\_UC404\_image0019.png]]

&#x20;

We can easily gain a root shell using GTFOBins:

!\[\[59\_UC404\_image0020.png]]

&#x20;

!\[\[59\_UC404\_image0021.png]]

&#x20;

Flag:

!\[\[59\_UC404\_image0022.png]]

3f57b2dd2884b71662538065adacedb2

&#x20;

Wasn't too bad...

&#x20;
