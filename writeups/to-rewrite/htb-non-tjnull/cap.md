# Cap

Cap

Tuesday, 29 March 2022

3:37 pm

A Linux machine with these ports open:

&#x20;

!\[\[13\_Cap\_image001.png]]

&#x20;

!\[\[13\_Cap\_image002.png]]

&#x20;

The FTP server does not allow for any anonymous logins. So off to the HTTP server we go.

&#x20;

!\[\[13\_Cap\_image003.png]]

This is all it has.

&#x20;

Looking around, the security snapshot would get us to download a pcap file.

Interestingly, the URL would put /1 there and the file downloaded would be 1.pcapng.

&#x20;

If we change it to 0, we get a different Pcapfile.

Upon analysing the paths, we get this.

!\[\[13\_Cap\_image004.png]]

We now have a password to work with.

&#x20;

With that, we can SSH in as the user.

&#x20;

!\[\[13\_Cap\_image005.png]]

&#x20;

When looking at the linpeas output:

!\[\[13\_Cap\_image006.png]]

&#x20;

We have this. This means we can run a script that would set uid as 0.

!\[\[13\_Cap\_image007.png]]

&#x20;

Simple.

&#x20;
