# DVR

DVR4

Saturday, 26 March 2022

4:44 pm

This is a Windows machine with these ports open.

!\[\[104\_DVR4\_image001.png]]

&#x20;

&#x20;

Port 8080 takes us to this argus surveillance page.

!\[\[104\_DVR4\_image002.png]]

Looks vulnerable, and searchsploit returns quite a few to choose from.

&#x20;

!\[\[104\_DVR4\_image003.png]]

&#x20;

There's this directory traversal one that we can use, and it indeed works.

!\[\[104\_DVR4\_image004.png]]

&#x20;

From here, we can think about a few files that would be good to exploit.

&#x20;

Interestingly, there's also this file whereby it was supposed to have credentials there.

!\[\[104\_DVR4\_image005.png]]

Not too sure what any of these mean.

&#x20;

From here, we can see the user, which is called viewer.

!\[\[104\_DVR4\_image006.png]]

&#x20;

Using the LFI, we can directly gain the user private SSH key.

!\[\[104\_DVR4\_image007.png]]

&#x20;

Funnily enough, we can cheese it and use this LFI to read the proof.txt, and this only indicates to me that we have to use argus again for privilege escalation as it seems to run as the administrator on the machine.

&#x20;

!\[\[104\_DVR4\_image008.png]]

SSH in and grab this flag.

&#x20;

Now, we need to capture the administrator flag legitimately.

Well, the privilege escalation method for Argus does not work as we do not have write access for that file.

&#x20;

!\[\[104\_DVR4\_image009.png]]

&#x20;

That undock one is new. But we are part of no groups here, so nothing funny. This would mean we have to either:

1. Hijack some DLL that uses Argus
2. Find some password

&#x20;

The latter was correct, as within the config file located here, we can find a password.

&#x20;

!\[\[104\_DVR4\_image0010.png]]

&#x20;

{width="6.583333333333333in" height="6.270833333333333in"}

This password0 was the random password I set when enumerating. From here, I reset the machine to make sure I didn't mess anything up.

!\[\[104\_DVR4\_image0012.png]]

There's also this exploit here.

&#x20;

Run the exploit and gain the password.

For now, we need to use the method whereby we run stuff using scriptblocks and the password given to us. There are two passwords in the file, so I'll be testing both.

&#x20;

We cannot SSH into the administrator, presumably because well, he doesn't have a .ssh file.

&#x20;

We can use the scriptblock method to begin execution of code as the administrator. Test it out using the whoami command first.

&#x20;

From there we can just grab a reverse shell as the administrator.

&#x20;

&#x20;

&#x20;

&#x20;

&#x20;

&#x20;

&#x20;
