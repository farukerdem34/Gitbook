# QuackerJack

QuackerJack

Wednesday, 16 March 2022

8:28 pm

The target IP is 192.168.159.57.

My IP is 192.168.49.159.

&#x20;

Enumeration!

!\[\[10\_QuackerJack\_image001.png]]

&#x20;

!\[\[10\_QuackerJack\_image002.png]]

FTP anonymous logins give nothing.

Port 80 is not interesting.

&#x20;

Port 8081 gives this:

!\[\[10\_QuackerJack\_image003.png]]

Certificate is not interesting.

&#x20;

!\[\[10\_QuackerJack\_image004.png]]

The version and stuff was more interesting.

&#x20;

The bottom exploit does not work properly.

The top exploit requires username and password.

&#x20;

Right, so we go to the web to search for exploits.

I found this one:

!\[\[10\_QuackerJack\_image005.png]]

&#x20;

Ran it, and we got a hash.

!\[\[10\_QuackerJack\_image006.png]]

&#x20;

Cracked this hash on crackstation.net.

!\[\[10\_QuackerJack\_image007.png]]

&#x20;

Logged in with admin as username.

{width="10.645833333333334in" height="2.6666666666666665in"}

&#x20;

Now we can take the authenticated exploit and run it.

&#x20;

!\[\[10\_QuackerJack\_image009.png]]

&#x20;

!\[\[10\_QuackerJack\_image0010.png]]

&#x20;

!\[\[10\_QuackerJack\_image0011.png]]

We are now this user called apache.

&#x20;

Checked linpeas:

!\[\[10\_QuackerJack\_image0012.png]]

&#x20;

!\[\[10\_QuackerJack\_image0013.png]]

&#x20;

!\[\[10\_QuackerJack\_image0014.png]]

Easy.

&#x20;

&#x20;
