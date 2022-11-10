# Sense

Sense

Tuesday, 1 February 2022

11:18 am

&#x20;

This is a Linux machine.

Target IP is 10.10.10.60.

Nmap scan shows that ports 80 and 443 are open, running lighttpd 1.4.35.

&#x20;

When going to the website, we can see a PFSense login page.

!\[\[07\_Sense\_image001.png]]

&#x20;

Further enumeration reveals that this webserver is running the PFSense engine, which has flaws pertaining to SQL Injection and Directory Traversal attacks.

&#x20;

Doing a quick SQLMap on the thing reveals nothing of interest.

&#x20;

Running a directory scan reveals nothing of interest. I decided to run **dirbuster** instead because it does more in-depth directory checking compared to that of the usual gobuster method.

&#x20;

!\[\[07\_Sense\_image002.png]]

&#x20;

This revealed some more interesting parts of the website, including that of a directory called under the system-users.txt as picked up from dirbuster.

!\[\[07\_Sense\_image003.png]]

&#x20;

A quick Google search also reveals that the default username of PFSense engines is 'pfsense'. With this, I managed to log in to the website and was able to see the version.

!\[\[07\_Sense\_image004.png]]

&#x20;

The version was 2.1.3. A quick searchsploit reveals an exploit.

!\[\[07\_Sense\_image005.png]]

Downloading the script and reading it, it seems we need to configure a listening port and it does it automatically kind of. I chose port 4444, and the exploit script is as shown below.

&#x20;

{width="11.541666666666666in" height="2.34375in"}

After setting the listening port and running the exploit, I was able to get a root shell and hence solve the challenge.

&#x20;

{width="10.833333333333334in" height="5.416666666666667in"}

&#x20;

All in all, an easier box to root. The hardest part was finding out details about the login. This required me to use Dirbuster, which performs more in-depth analysis of the files.

&#x20;

Lessons learnt:

1. Do not store any form of details on the web server, **especially any form of user credentials.** If we must store it, please do so behind the DMZ in databases that are secured.
