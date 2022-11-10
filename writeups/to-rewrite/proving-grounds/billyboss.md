# Billyboss

Billyboss

Monday, 21 March 2022

6:07 pm

Nmap scan reveals these:

!\[\[98\_Billyboss\_image001.png]]

&#x20;

From here, port 80 is not interesting, FTP does not yield anything.

&#x20;

Port 8081 however, does yield this one interesting tidbit.

!\[\[98\_Billyboss\_image002.png]]

This version has an RCE for it, and the default credentials of nexus nexus works on this server.

&#x20;

!\[\[98\_Billyboss\_image003.png]]

Get a rev.exe onto it and execute it using the script. We would get a shell back.

&#x20;

!\[\[98\_Billyboss\_image004.png]]

&#x20;

Interestingly, there's this little bit here:

!\[\[98\_Billyboss\_image005.png]]

&#x20;

Seems that we can run commands in this portion of our desktop under that name.

JuicyPotato.exe does not work despite being windows 10 pro with the privilege misconfigurations.

{width="6.645833333333333in" height="0.7291666666666666in"}

&#x20;

{width="10.354166666666666in" height="0.7916666666666666in"}

&#x20;

!\[\[98\_Billyboss\_image008.png]]

&#x20;

!\[\[98\_Billyboss\_image009.png]]

&#x20;

Anyways, the most important thing is the SeImpersoantePrivilege token being set to true. We can use the PrintSpoofer.exe to spawn a new administrative shell.

{width="5.96875in" height="2.6041666666666665in"}

&#x20;

!\[\[98\_Billyboss\_image0011.png]]

&#x20;

&#x20;

&#x20;
