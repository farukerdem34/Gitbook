# Medjed

Medjed

Friday, 18 March 2022

3:26 pm

Confusing box with loads of services running on it.

&#x20;

Funnily enough, we can grab both the flags through the use of the file server. But that isn't true pentesting.

&#x20;

From the web file server, we can actually just upload files directly into the machine.

&#x20;

Anyways, look at port 33033, which has a login page that we can bypass through directory changing. There is one /slug directory which would let us access this page that is vulnerable to SQLi.

&#x20;

We can use this to upload some cmd.php stuff, then msfvenom for the exe file for a shell back.

!\[\[94\_Medjed\_image001.png]]

From there, there are some unquoted service paths, and we can plant shells there and then restart the system.

!\[\[94\_Medjed\_image002.png]]

&#x20;

&#x20;

&#x20;
