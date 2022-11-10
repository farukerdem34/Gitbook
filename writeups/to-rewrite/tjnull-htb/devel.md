# Devel

Devel

Tuesday, 1 February 2022

9:45 pm

&#x20;

> The target IP address is 10.10.10.5.
>
> My IP is 10.10.17.233.
>
> This is a Windows machine we are targetting.
>
> &#x20;
>
> Nmap Scan
>
> !\[\[09\_Devel\_image001.png]]
>
> &#x20;
>
> Nmap reveals one FTP server and one web server. The versions of the ftp server has been hidden perhaps intentionally.
>
> Further scanning reveals nothing about higher level ports and stuff.
>
> &#x20;
>
> Visiting the web page seems to reveal that this is an IIS7 web server.
>
> No directories were revealed with the HTTP server.
>
> &#x20;
>
> !\[\[09\_Devel\_image002.png]]
>
> &#x20;
>
> A quick searchsploit reveals that there are exploits related to IIS 7.5, which is the version being used in this case. There are multiple vulnerabilities and stuff regarding this, all of which are not notable.
>
> &#x20;
>
> Moving on back to FTP, we can do some banner grabbing to determine that this is Microsoft FTP Service running. Interestingly, when enumerating further, I saw this interesting piece regarding the Nmap scan into port 21. It allows anonymous logins!
>
> &#x20;
>
> !\[\[09\_Devel\_image003.png]]
>
> &#x20;
>
> This would mean that we are able to see what type of files are stored on the FTP server. We can do so using the username:password **anonymous:anonymous**.
>
> &#x20;
>
> &#x20;
>
> &#x20;
>
> From there, we can list out all the possible files using **ls -a**. From here, we can clearly see that we are within the website directory, and that any file uploaded here can be abused.
>
> !\[\[09\_Devel\_image004.png]]
>
> &#x20;
>
> !\[\[09\_Devel\_image005.png]]
>
> &#x20;
>
> From here, it seems that the files are rather boring and has nothing of use. I began to wonder if it's possible to upload or download some form of backdoor into the system.
>
> &#x20;
>
> Anyways, we are also able to use our web browser to connect to the FTP server. This can be done through searching up [**ftp://anonymous:anonymous@10.10.10.5**](ftp://anonymous:anonymous@10.10.10.5). Nothing changes in terms of files, but still interesting nonetheless!
>
> &#x20;
>
> Anyways it seems that we would need to upload a file somehow to the server and I think we can execute it, seeing as that we're literally in the website directories and these are being executed when handling the website. The website seems to be running aspnet, hence I would need a .ASPX shell. I will try to upload this onto the website and see if I can get a shell from it. Anyways, I used some MSFVenom-fu to generate a payload.
>
> &#x20;
>
> {width="15.416666666666666in" height="2.9270833333333335in"}
>
> Transfer that straight to the FTP server directory.
>
> &#x20;
>
> {width="9.739583333333334in" height="4.364583333333333in"}
>
> &#x20;
>
> Opening a listener port works! Just like that we have a shell to work with. When doing whoami, I can see that we are indeed the website iis apppool\web.
>
> &#x20;
>
> {width="14.010416666666666in" height="3.0729166666666665in"}
>
> &#x20;
>
> Afterwards, trying to access any User file is not allowed. Time for privilege escalation. Windows privilege escalation has methods like WinPEAS, a script that does finds possible vectors to make use of. In this case, I wanted to do it manually. Sometimes, we can attempt to use it, but perhaps the machine is configured to not let anyone execute, and more importantly in this case, **we are still just the application and not even a user**. I felt that the likelihood of it succeeding is low, and I'm already forcing myself not to use MSF so might as well stick by it!
>
> &#x20;
>
> These are the steps I did to check on what I can do:

1. Check IP routing tables and stuff (Nothing interesting)
2. Check the version (**Windows 7 Enterprise, 32-bit**)
3. Check all accounts that are available (Administrator, babis, Guest)
4. Check the firewall (through the use of netsh firewall show state, of which case I did not find anything of use)
5. **Check the privileges that we have (done through whoami /priv)**

> &#x20;
>
> In this case, I went with the last option. We can see that we are allowed to SeImpersonatePrivilege. Some google-fu led me to the Juicy Potato exploit! This exploit also works on Windows 7 Enterprise, 32-bit.
>
> &#x20;
>
> !\[\[09\_Devel\_image009.png]]
>
> &#x20;
>
> This makes use of those facts to elevate privileges. Anyways to use juicy potato, we need a reverse shell file. Using msfvenom to generate a quick Windows reverse shell again and transfer it to the server. After downloading Juicy Potato 32-bit systems, we transfer it to the servers as follows:
>
> &#x20;
>
> !\[\[09\_Devel\_image0010.png]]
>
> &#x20;
>
> After getting both Juicy.exe and reverse.exe into the same place, we need to use JuicyPotato to identify a suitable token that we can use in our exploit. This can be done with the following command:
>
> &#x20;
>
> {width="15.791666666666666in" height="1.7291666666666667in"}
>
> &#x20;
>
> After we have identified this, we can safely use our reverse shell (on port 1337) to execute the Juicy file and get us a shell!
>
> &#x20;
>
> {width="15.010416666666666in" height="2.53125in"}
>
> &#x20;
>
> {width="8.635416666666666in" height="2.8333333333333335in"}
>
> &#x20;
>
> Challenging box to me, best part was not referring to the answers.. Was really fulfilling. I need to learn more on Windows privilege escalations. It's very clear that this
>
> &#x20;
>
> Lessons learnt for security:

1. **Do not allow for anonymous login on FTP Servers.** This was one major flaw in the security, because it allowed us to start transferring files like backdoors and payloads into it to gain an initial shell.
2. **Set the privileges of all accounts properly.** This would include that of the webpool application, because I was able to have some permissions in areas, it led to the use of Juicy Potato to exploit and gain a shell.
3. There is also a need to limit powershell usage and general ability to browse directories for every single possible user. This acts as defence in-depth. Should anyone gain a shell into our system, there would be sure fire methods to prevent escalation
4. Perhaps the most important, **SOFTWARE UPGRADES.** Windows 7 has a plethora of vulnerabilities, and one can pivot and take advantage of these.

> &#x20;
>
> &#x20;
>
> &#x20;
>
> &#x20;
>
> &#x20;
>
> &#x20;
>
> &#x20;
>
> &#x20;
>
> &#x20;
>
> &#x20;
