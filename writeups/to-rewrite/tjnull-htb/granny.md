# Granny

Granny

Sunday, 6 February 2022

2:01 pm

This is a Windows machine.

Target IP is 10.10.10.15.

My IP is 10.10.16.5

&#x20;

Nmap scan with vuln script.

&#x20;

!\[\[17\_Granny\_image001.png]]

&#x20;

Seems that there are a lot of exploits related to this. Anyways all the directories I found using gobuster and dirbuster were not useful.

!\[\[17\_Granny\_image002.png]]

&#x20;

Anyways a bit of googling reveals that there are some vulnerabilities associated with IIS 6.0, which is a really old version of the software. We can use davtest, a tool used for really old websites.

&#x20;

!\[\[17\_Granny\_image003.png]]

&#x20;

Seems that we can literally put stuff on the website, so let's try it with a reverse php shell. Davtest is the enumeration tool, and the next tool would be cadaver.

&#x20;

Found out that PHP does not work, hence we need to use aspx because I forgot this is an aspnet\_client. It seems that there are firewalls preventing us from uploading this onto the page. According to davtest, we need to follow the allowed configurations in order to upload our web shell. I changed the current file to a text file.

&#x20;

Too much effort. I found another [IIS 6.0 reverse shell](https://github.com/g0rx/iis6-exploit-2017-CVE-2017-7269/blob/master/iis6%20reverse%20shell) that could be used.

&#x20;

!\[\[17\_Granny\_image004.png]]

Run it, and there we go we have a shell.

!\[\[17\_Granny\_image005.png]]

&#x20;

Doing a bit of banner grabbing, and found out that this is a really old Windows 2003 Server, 32-bit and also has some hotfixes. Means no juicypotato for me.

{width="9.375in" height="7.395833333333333in"}

&#x20;

Found out that the churrasco.exe exploit works the best amongst all.

&#x20;

Also found out that older Windows machines do not have powershell or certutil downloaded onto it, hence I resorted to looking at solutions for vbscript.

&#x20;

!\[\[17\_Granny\_image007.png]]

&#x20;

Did basically this and managed to transfer some files over.

!\[\[17\_Granny\_image008.png]]

&#x20;

A listener port should receive the root shell.

&#x20;

This was a useful little box, as I managed to learn a bit more about vbscript, and that it's really useful should wget, certutil and powershell all fail me. Just another thing to add to tricks up my sleeves.

&#x20;

Also good to take note that the usage of older web servers is terrible and that we are able to change the directories.

Learnt some new tools in the form of cadaver and davtest too, even though I did not end up using them.

&#x20;

&#x20;
