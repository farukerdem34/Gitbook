# Jerry

Jerry

Tuesday, 8 February 2022

12:53 pm

This is a Windows machine.

My IP is 10.10.16.5.

The target IP is 10.10.10.95.

&#x20;

Enumeration.

Nmap scan reveals port 8080 is open, and it's running Apache Tomcat/Coyote JSP engine 1.1.

!\[\[24\_Jerry\_image001.png]]

&#x20;

Cool. Ran a directory scan to check on what's up.

&#x20;

!\[\[24\_Jerry\_image002.png]]

&#x20;

Not much from here.

Anyways, it's running Tomcat/7.0.88 as shown on the website.

!\[\[24\_Jerry\_image003.png]]

&#x20;

There are some exploits for this.

Let's begin by looking at the directories that were enumerated.

&#x20;

Went to /manager and found this page with a password on it!

!\[\[24\_Jerry\_image004.png]]

&#x20;

For some reason this seems to have blocked me or something, so I restarted the machine.

&#x20;

For some reason it does not work at all, does not let me log in.

&#x20;

Resorted to using MSF LOL.

&#x20;

!\[\[24\_Jerry\_image005.png]]

&#x20;

Not a hard box.

&#x20;

&#x20;
