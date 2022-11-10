# Blue

Blue

Tuesday, 1 February 2022

1:22 am

&#x20;

The target IP is **10.10.10.40**, and my Kali machine IP is **10.10.16.3**.

&#x20;

This machine is a Windows machine and has to do with patch management.

&#x20;

Enumeration!

!\[\[06\_Blue\_image001.png]]

&#x20;

Interestingly, there are lot of RPC ports open and the usual SMB ports open.

Doing a list using SMBclient leads us to find out there is a 'Users' and 'Share' share available.

&#x20;

!\[\[06\_Blue\_image002.png]]

&#x20;

Connecting to 'Share' yields no interesting results.

!\[\[06\_Blue\_image003.png]]

&#x20;

&#x20;

Connecting to Users yields some interesting results when looking around.

&#x20;

&#x20;

When in \Default, I have found some NTUSER.DAT files, which are the files that contain the stuff regarding profiles of the users that can access the computer. Getting some of these files and analysing them reveals nothing interesting, we are unable to understand what they are saying.

&#x20;

Analysing the original Nmap again, we only can know that this is PC is called HARIS-PC and that's about it. I attempted using an the nmap script 'vuln' to further enumerate any possible vulnerabilities. This returned the fact that the servers within the machine were vulnerable to ms17-010, the EternalBlue exploit.

&#x20;

!\[\[06\_Blue\_image004.png]]

&#x20;

Running a searchsploit to understand which of the ones we are going to use is key, and we know that this machine possibly uses Windows 7-10, hence we are going to take 42315.py and attempt with that first. This version of the exploit covers multiple different versions.

&#x20;

This is a basic method of which EternsasalBlue can be exploited, and I will not go into further details regarding the automated way.

&#x20;

Lessons learnt:

1. Please do not let EternalBlue affect you.

&#x20;
