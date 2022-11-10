# Mice

Mice

Saturday, June 18, 2022

10:52 AM

Nmap scan:

!\[\[109\_Mice\_image001.png]]

&#x20;

!\[\[109\_Mice\_image002.png]]

&#x20;

&#x20;

Port 1978:

This has a weird thing upon connection as pointed out by Nmap.

&#x20;

Turns out there is an exploit for this, called Remote Mouse 3.08 that is exploitable.

!\[\[109\_Mice\_image003.png]]

&#x20;

{width="9.0625in" height="2.1354166666666665in"}

&#x20;

The PoC spawns calc.exe, hence we need to find one that would execute some form of command.

[https://github.com/p0dalirius/RemoteMouse-3.008-Exploit/blob/master/RemoteMouse-3.008-Exploit.py](https://github.com/p0dalirius/RemoteMouse-3.008-Exploit/blob/master/RemoteMouse-3.008-Exploit.py)

&#x20;

We can just change a few parameters here and there to download nc.exe onto the server and then execute it accordingly.

&#x20;

Then, we would gain a reverse shell.

!\[\[109\_Mice\_image005.png]]

&#x20;

{width="7.145833333333333in" height="1.7708333333333333in"}

&#x20;

Flag:

!\[\[109\_Mice\_image007.png]]

Fc2f78a044111a4df8f4f07e309db265

&#x20;

PE:

!\[\[109\_Mice\_image008.png]]

&#x20;

There's this FileZilla FTP Client present.

&#x20;

WinPEAS:

!\[\[109\_Mice\_image009.png]]

&#x20;

!\[\[109\_Mice\_image0010.png]]

&#x20;

There's this base64 password here.

{width="4.90625in" height="0.90625in"}

&#x20;

Seems that we do have some credentials for the FileZilla server.

&#x20;

The user we are in as can also RDP in:

!\[\[109\_Mice\_image0012.png]]

&#x20;

RDP in:

{width="6.8125in" height="0.75in"}

&#x20;

Since we have Remote Mouse 3.008, this is vulnerable to a Local Privilege Escalation exploit.

!\[\[109\_Mice\_image0014.png]]

[https://www.exploit-db.com/exploits/50047](https://www.exploit-db.com/exploits/50047)

&#x20;

Following the exploit, we can just open RemoteMouse using this arrow.

!\[\[109\_Mice\_image0015.png]]

&#x20;

{width="3.6354166666666665in" height="5.03125in"}

&#x20;

Press Change and enter C:\Windows\system32\cmd.exe and an admin shell will spawn.

{width="5.3125in" height="1.8333333333333333in"}

&#x20;

Flag:

!\[\[109\_Mice\_image0018.png]]

7f8f5660286f6b1396c109d7cc5bdd39

&#x20;

&#x20;
