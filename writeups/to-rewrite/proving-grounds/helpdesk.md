# Helpdesk

Helpdesk

Tuesday, 15 March 2022

12:06 am

We start with this Windows machine.

The target IP is 192.168.73.43.

My IP is 192.168.49.73.

&#x20;

This is an easy rated box, that should get us into check of what is the standard of PG machines. I pray the easy here isn't the medium level of HTB.

&#x20;

Nmap this. Takes quite a while to scan.

Took so long that I just used Kali within the website...

&#x20;

Nmap still did not work, and I was getting really impatient. I just looked up a guide and then took his Nmap scan.

&#x20;

!\[\[82\_Helpdesk\_image001.png]]

Alright, we have a web server.

&#x20;

Finally...

Now, we can proceed to try and get into this service.

!\[\[82\_Helpdesk\_image002.png]]

Okay, there's this thing here.

&#x20;

The default credentials of administrator:administrator works.

&#x20;

Now, we are in.

&#x20;

!\[\[82\_Helpdesk\_image003.png]]

Looking at that version of 7.6.0, I ran a searchsploit on my main kali machine.

&#x20;

!\[\[82\_Helpdesk\_image004.png]]

There's a whole lot of vulnerabilities, but none of them are interesting in my case.

&#x20;

When trying to view something, I checked that this was indeed tomcat.

!\[\[82\_Helpdesk\_image005.png]]

This version in specific.

!\[\[82\_Helpdesk\_image006.png]]

Probably could exploit these somehow.

&#x20;

Googling about the engine and for exploits, there was this additional puzzle to solve:

!\[\[82\_Helpdesk\_image007.png]]

I wish this was my own VM. I miss it.

&#x20;

Found this exploit which basically gives me the shell I want.

&#x20;

!\[\[82\_Helpdesk\_image008.png]]

Simple enough.

&#x20;

Now, we just need to run this thing, and generate a java shell using a msfvenom, which should be simple enough I mean...

&#x20;

Wot.

!\[\[82\_Helpdesk\_image009.png]]

&#x20;

OK...MSF time.

!\[\[82\_Helpdesk\_image0010.png]]

Use this, and we can get a meterpreter shell.

&#x20;

!\[\[82\_Helpdesk\_image0011.png]]

As the admin. End of box.

&#x20;

In true OSCP fashion, we need to provide the output of ipconfig and the flag in one screenshot:

!\[\[82\_Helpdesk\_image0012.png]]

MSF was used because I'm tired!

&#x20;

There we go, rooted.

I sincerely pray and hope that the connection gets better.

&#x20;

&#x20;

&#x20;

&#x20;

&#x20;
