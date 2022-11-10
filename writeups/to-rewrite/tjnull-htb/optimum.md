# Optimum

Optimum

Wednesday, 2 February 2022

5:00 pm

This machine is a Windows machine.

My IP address is 10.10.17.233.

The target IP is 10.10.10.8.

&#x20;

Nmap scan is always up first. The basic nmap scan reveals port 80 is open on it, and I did a more thorough scan through nmap -sV -O -T4 -p- 10.10.10.8.

Seems that there is only a HFS2.3 server running, of which I know there's a vulnerability for it already.

!\[\[11\_Optimum\_image001.png]]

&#x20;

Searchsploit reveals there are several RCEs that affect this version of HFS.

&#x20;

!\[\[11\_Optimum\_image002.png]]

&#x20;

Viewing the web page, it seems to be a simple server of which we can login and browse files and stuff. I suppose we need to execute a reverse shell somehow through this method.

&#x20;

Anyways, I tried two different exploits, 49584.py and 39161.py. 49584.py works to give me a user shell. Be sure to change the IP addresses listed within the exploit.

!\[\[11\_Optimum\_image003.png]]

&#x20;

{width="5.84375in" height="6.28125in"}

&#x20;

Privilege escalation time. Take note that this is a 64-bit machine when using systeminfo to check. In this case, I wanted to try out winPEAS.exe. Download the 64 bit version and use a HTTP server to bring it over. However, this seems to be blocked by some firewall, because it just doesn't work. Seems that all HTTP methods are being blocked, and I can't get any files onto the host machine.

&#x20;

I assume this is a bug of some sort, because I have checked a dozen times regarding the code after giving up up and wondering why it is not working.

&#x20;

Anyways, the winPEAS would detect MS16-098 exploit, which would give us root privileges. This would allow us to get the root flag.

&#x20;

First time encountering issues with the transferring of files and stuff to the host machine. Either ways, it was a simple procedure forward.

&#x20;

Things I learnt:

* Banner grabbing is great
* A bit more on how to transfer files, Google-fu led me to SMB methods and powershell using HTTP servers set up on our Kali machine.

&#x20;

Security stuff:

1. Update the OS and things
2. Update your machine to prevent being exploited by such privilege escalation attacks.

> &#x20;
