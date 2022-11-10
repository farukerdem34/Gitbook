# Jacko

Jacko

Monday, 21 March 2022

12:18 pm

This is a classic example of H2 RCE, as the version of 1.4.199 is a vulnerable version of this.

&#x20;

!\[\[96\_Jacko\_image001.png]]

From there, we can basically run an SMB server and execute nc.exe from the SMB root folder.

From here, we can see that the whoami command does not work. Perhaps the PATH is out of order.

&#x20;

!\[\[96\_Jacko\_image002.png]]

&#x20;

!\[\[96\_Jacko\_image003.png]]

Just need to change it to system32 and the cmd.exe shell would work.

!\[\[96\_Jacko\_image004.png]]

&#x20;

Downloaded winPEAS into this and ran it to check for vulnerable.exe files that could be abused for us.

!\[\[96\_Jacko\_image005.png]]

&#x20;

Alright, then we can find this file here.

&#x20;

!\[\[96\_Jacko\_image006.png]]

This has an exploit for it.

&#x20;

!\[\[96\_Jacko\_image007.png]]

When run however, it does not work for me. Since I executed it according to the guide, and it still does not work, and after resorting to looking at writeups for the machine, I decided to leave this one alone.

&#x20;

Done for now I guess.

&#x20;

&#x20;
