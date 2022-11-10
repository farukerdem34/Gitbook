# Bastard

Bastard

Friday, 4 February 2022

10:24 am

This is a Windows machine.

The target IP is 10.10.10.9.

My IP is 10.10.17.233

&#x20;

Nmap scan first.

!\[\[14\_Bastard\_image001.png]]

&#x20;

&#x20;

Ran a few more Nmap scans, particularly one using the vuln script that nmap has. Also ran a gobuster to see what else this thing has. However, I think the web server has blocked some stuff regarding directory scans, because I was unable to make the scan run faster. It ran like 6/requests a second.

!\[\[14\_Bastard\_image002.png]]

&#x20;

!\[\[14\_Bastard\_image003.png]]

&#x20;

These are all interesting information. Seems to point toward the fact that there is a login page within the web directories. Did one more scan just to fully enumerate Port 80 on the machine.

&#x20;

Anyways, I visited the webpage, and there's a login present. Its powered by Drupal. Drupal has a crap load of vulnerabilities through using searchsploit.

!\[\[14\_Bastard\_image004.png]]

&#x20;

Drupal 7.0 has a lot of exploits, but it's not clear which one works as of now. The earlier nmap scans show that there is a robots.txt file on the web server. This presents us with a lot of directories to view through. Looking through some of them, there is interesting information within the CHANGELOG.txt file.

!\[\[14\_Bastard\_image005.png]]

&#x20;

So this web server is running Drupal 7.54.

Doing a searchsploit on Drupal 7 gives me a load of vulnerabilities.

> !\[\[14\_Bastard\_image006.png]]

&#x20;

Doing a search on the individual version, which is 7.54, there's an RCE called Drupalgeddon3.

!\[\[14\_Bastard\_image007.png]]

&#x20;

I went with 44449, the only one I could make work. Downloaded highline via **sudo gem install highline**. While waiting for the exploit to run, I was thinking of privilege escalations, in this case because it's a Windows IIS server. It seems to be running a legacy version of Windows, hence I had things like winPEAS.exe and JuicyPotato.exe prepared.

&#x20;

The exploit takes a while to run, so just leave it while it does its thing.

&#x20;

!\[\[14\_Bastard\_image008.png]]

From here, we can just grab that user.txt

&#x20;

Next is privilege escalation. What I first did was banner grab some stuff.

!\[\[14\_Bastard\_image009.png]]

&#x20;

We can see that this is a Windows Sever 2008 R2 Datacenter, and it does not have any hotfixes and stuff. Some Google-fu led me to MS15-051. This is an exploit that works on the current version of the windows server.

!\[\[14\_Bastard\_image0010.png]]

&#x20;

Get the 64-bit version and transfer it over. Execution makes us admin! We just need to get out that flag somehow. Trying to do so with this method does not seem to work.

&#x20;

I googled and we need to install nc64.exe onto the system as well. This would make it such that we are able to get a shell through nc64.exe
