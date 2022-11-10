# LogForge (WIP)

LogForge (WIP)

Thursday, July 28, 2022

7:47 PM

Nmap scan:

!\[\[38\_LogForge (WIP)\_image001.png]]

&#x20;

!\[\[38\_LogForge (WIP)\_image002.png]]

&#x20;

Good to know.

Detailed nmap:

Nothing.

&#x20;

Gobuster:

!\[\[38\_LogForge (WIP)\_image003.png]]

&#x20;

!\[\[38\_LogForge (WIP)\_image004.png]]

&#x20;

Can see that there are some other directories present.

When viewing images, we can see that we are greeted by a Tomcat 9.0.31 server.

!\[\[38\_LogForge (WIP)\_image005.png]]

&#x20;

We would need a method of which we can access the admin or manager directories, because that's the only thing we can find from here.

Hacktricks has a section on directory traversal with the ; character.

!\[\[38\_LogForge (WIP)\_image006.png]]

&#x20;

With this, we can actually log in.

We can login with tomcat:tomcat, and from here, we can access the dashboard.

!\[\[38\_LogForge (WIP)\_image007.png]]

&#x20;

Then we just need to include the WAR file we want to upload.

!\[\[38\_LogForge (WIP)\_image008.png]]

&#x20;

However, we are unable to upload this.

!\[\[38\_LogForge (WIP)\_image009.png]]

&#x20;

So if we cannot gain a shell through WAR files, then we would need to find other exploits regarding this.

Since the box is called logforge, there is the popular log4shell exploit, which is an RCE exploit for Java based applications, like Tomcat.

This was a WIP, as exploiting log4shell was something I was not familiar with.

&#x20;

&#x20;
