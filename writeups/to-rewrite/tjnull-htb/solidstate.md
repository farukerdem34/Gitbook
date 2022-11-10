# SolidState

SolidState

Tuesday, 1 February 2022

12:51 pm

&#x20;

This is a Linux machine, and it has to do with web and file misconfigurations.

The target IP is 10.10.10.51.

&#x20;

Nmap scan first up.

!\[\[08\_SolidState\_image001.png]]

&#x20;

Interesting, I see the mail stuff open. It seems that JAMES is the name of the server or something. Take note SSH is open!

&#x20;

Did a more in-depth scan of the port 25. For context, SMTP (Simple Mail Transfer Protocol) and POP3 are mail ports that would allow for the receiving and sending of mails for servers. The common security vulnerabilities that are included within these are open relay attacks, where attackers can spoof spam emails through port 25 to make it seem like it's coming from a legit place.

&#x20;

Port 119 on the other hand, is the Network News Transfer Protocol, which is commonly used to transfer Usenet news articles between new servers (Wikipedia).

&#x20;

Using Telnet to connect to port 25 would reveal nothing of interest, as no commands were specified from Nmap. From here, I attempted banner grabbing the version of the version of POP3 port. It seems that the Host is named 'solidstate'. After some poking around with Nmap, I found a port 80 open that was previously undetected.

&#x20;

!\[\[08\_SolidState\_image002.png]]

&#x20;

Connecting to it reveals some type of website, of which the contents are not interesting or noteworthy. Nice looking website though!

!\[\[08\_SolidState\_image003.png]]

Running a dirbuster on it to reveal the full tree! Using the following command so I can do a thorough analysis of what is present.

&#x20;

The web engine seems to be using HTML5 UP, when looking at the main.js file dirbuster found for me. There is some form of HTML5 Wordpress exploits I can find using searchsploit, but it does not seem relevant. Dirbuster was unable to find anything else besides /assets and /images.

!\[\[08\_SolidState\_image004.png]]

Digging around the assets more, I found out the specific engine it uses is HTML5 Shiv v3.6.2. This is under the /assets directory of the website.

&#x20;

!\[\[08\_SolidState\_image005.png]]

&#x20;

All in all, there seems to be NOTHING on the HTTP server hosted. Back to SMTP.

&#x20;

I checked that the SMTP was running on JAMES v2.3.2. A quick searchsploit reveals that this has an RCE exploit!

&#x20;

!\[\[08\_SolidState\_image006.png]]

&#x20;

Reading the script from the 35513.py exploit, it seems to connect to the web server using netcat. The password and username seem to be root and root. Doing so leads me to some kind of CLI.

{width="11.802083333333334in" height="6.416666666666667in"}

&#x20;

{width="11.9375in" height="6.9375in"}

Doing the listusers command, I can see that there are 6 users (1 from trying the other exploit, as you can see it launched a payload into it...) . Since we are root, we can set passwords for these. I set the password for all users to be 1234 from there.

&#x20;

Ok anyways, this password is not for SSH, but rather for the mail server held on port 110. I was able to log in to every user, and using some Google-fu to determine what are the commands that we can execute on them. I tried Mindy first, and I seem to have gotten some password and username for a shell! Using SSH, I can find the first flag.

&#x20;

{width="11.78125in" height="5.8125in"}

&#x20;

{width="7.020833333333333in" height="7.802083333333333in"}

&#x20;

Now it's time to privilege escalate to root somehow. Seems that for mindy, everything is restricted besides my own directory. This shell is very restricted, and I googled how to escape rbash shells. Interestingly, I found this command:

**Ssh user@ip -t "bash --noprofile"**

This gave me a small account privilege, as in I change directory and stuff.

I had to refer to the solutions at this point, because I did not know how to priv escalate.

&#x20;

!\[\[08\_SolidState\_image0011.png]]

&#x20;

Anyways, this brought me to download pspy, which is a script that allows for us to monitor what files are being executed. We can notice there is a /opt/tmp.py being executed sometimes. As such, we can append code into that file to give us a shell.

&#x20;

!\[\[08\_SolidState\_image0012.png]]

&#x20;

!\[\[08\_SolidState\_image0013.png]]

&#x20;

!\[\[08\_SolidState\_image0014.png]]

&#x20;

&#x20;

Starting a listening port and waiting for the file to be executed, we can get a shell which is root.

&#x20;

{width="11.21875in" height="3.5520833333333335in"}

&#x20;

Lessons Learnt:

1. Never, **NEVER**, use default credentials for anything regarding a server. The initial exploit was possible as we could spoof the root account through the default credentials.
2. There was a misconfiguration of the security settings, allowing us to execute the PSPY script and monitor what's going on within the system.
3. Do not let anyone append or alter files that the root account is executing.

&#x20;

Overall, fun box. Learnt some new things like LinEnum.sh as well as Pspy. Both are tools I will definitely use again.

&#x20;

&#x20;

&#x20;

&#x20;

&#x20;

&#x20;
