# Mirai

Mirai

Tuesday, 8 February 2022

7:41 pm

This is a Linux machine.

The target IP is 10.10.10.48.

My IP is 10.10.16.5.

&#x20;

Fast check reveals DNS, SSH and HTTP open on the web server.

The machine is tagged with Linux and Forensics. Expect to look through some folder here to get what we want.

&#x20;

!\[\[26\_Mirai\_image001.png]]

&#x20;

There are 2 http servers that are open, so take note. Enumerate both of their directories present to see what we can find.

So far I know there's an /admin penal at the web page of the port 80 server.

!\[\[26\_Mirai\_image002.png]]

&#x20;

Connecting to admin gives us this page, with a login page.

!\[\[26\_Mirai\_image003.png]]

&#x20;

Pi-hole is a web server that is basically an ad blocker.

We can take note of the version of Pi-Hole and stuff at the bottom.

&#x20;

!\[\[26\_Mirai\_image004.png]]

&#x20;

A very quick metasploit search reveals there are RCEs possible on this, should we be authenticated.

&#x20;

!\[\[26\_Mirai\_image005.png]]

&#x20;

The other webserver also has another log in page.

!\[\[26\_Mirai\_image006.png]]

&#x20;

Somehow we need to get into this. PLEX Server is a media player server with a player application in it. This is likely the next step because we cannot do anything else other than this.

&#x20;

There is no way of which we can even sign up for anything as well, due to the server stating to have an error when connecting to Plex.

&#x20;

Anyways, the default user credentials for this is **pi:raspberry**. This guy on Reddit told me.

{width="9.166666666666666in" height="2.6875in"}

&#x20;

SSH using that, and we're in.

&#x20;

{width="10.3125in" height="5.645833333333333in"}

&#x20;

Grab the user flag and it's time to privilege escalate.

Got LinEnum.sh onto this.

Revealed this.

!\[\[26\_Mirai\_image009.png]]

&#x20;

Seems that our user can run sudo su. We have the password.

&#x20;

!\[\[26\_Mirai\_image0010.png]]

I thought that was it. But the creator decided to be a little funny.

!\[\[26\_Mirai\_image0011.png]]

By the way, the flag is on /dev/sdb. We have to strings it. This is because we may delete the file, strings can read it due to the fact that Unix OS(s) process everything like a file.

&#x20;

**Strings /dev/sdb** would find us the file. No point going into a rabbit hole on a laggy machine after we basically rooted it.

&#x20;
