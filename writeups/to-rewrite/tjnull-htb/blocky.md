# Blocky

Blocky

Wednesday, 9 February 2022

12:36 pm

This is a Linux machine.

My IP is 10.10.16.5.

The target IP is 10.10.10.37.

&#x20;

Nmap scan reveals this:

!\[\[29\_Blocky\_image001.png]]

&#x20;

Sophos is just a remote management system, which is not my concern here.

&#x20;

Directory scan reveals LOTS of wordpress related things. WPScan it is.

&#x20;

!\[\[29\_Blocky\_image002.png]]

&#x20;

There's also a phpmyadmin page.

&#x20;

!\[\[29\_Blocky\_image003.png]]

&#x20;

!\[\[29\_Blocky\_image004.png]]

&#x20;

WpScan reveals this is a vulnerable version of wordpress.

!\[\[29\_Blocky\_image005.png]]

&#x20;

There are 2 users identifed by this.

!\[\[29\_Blocky\_image006.png]]

Ran a password cracker with rockyou.txt, for now let's move on to exploring more.

&#x20;

There are two mystery .jar files within /plugins.

&#x20;

!\[\[29\_Blocky\_image007.png]]

Examination of BlockCore reveals that there is a sqluser, called root and the password.

&#x20;

!\[\[29\_Blocky\_image008.png]]

&#x20;

I noted that now I have 3 users available for me.

&#x20;

I have notch, Notch and root.

And one password '8YsqfCTnvxAUeduzjNSXe22'.

&#x20;

I also know that SSH is present on the box, so let's try it with all the users.

Tried it with notch@10.10.10.37, and it worked!

&#x20;

!\[\[29\_Blocky\_image009.png]]

&#x20;

A quick sudo -l reveals that I can run all commands.

{width="10.84375in" height="1.4375in"}

&#x20;

That was easy lol.

&#x20;

!\[\[29\_Blocky\_image0011.png]]

The best part was that there was an actual minecraft server on the home directory of the user, which was funny.
