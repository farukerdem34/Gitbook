# Dibble

Dibble

Tuesday, 22 March 2022

4:48 pm

A Linux machine with these ports:

!\[\[30\_Dibble \_image001.png]]

Anonymous FTP logins work. However, it's giving me the hanging thing and I can't get any further.

&#x20;

!\[\[30\_Dibble \_image002.png]]

Drupal9! Means we won't be exploiting that page for now.

&#x20;

!\[\[30\_Dibble \_image003.png]]

Mongodb is here too.

&#x20;

On port 3000, there's this interesting log here.

!\[\[30\_Dibble \_image004.png]]

&#x20;

This seems to be some kind of commit or something.

&#x20;

Interesting, however, I was at a wall here on what to do. The FTP server had my suspicions, as I'm frequently unable to access that thing due to how Off Sec VPNs work for me.

&#x20;

Anyways, continuing now!

When looking at one of the requests, this comes across as weird to me.

!\[\[30\_Dibble \_image005.png]]

User level looks like base64.

&#x20;

!\[\[30\_Dibble \_image006.png]]

Right, so from here, we should be able to replace this with an admin cookie.

&#x20;

Once we have done that, we come to this Event Message portion, whereby technical details can be sent if needed?

!\[\[30\_Dibble \_image007.png]]

Perhaps this is an RCE vector, that we can use to get a reverse shell or something.

After getting the request, we can make sure to change the cookie to become an admin user.

&#x20;

!\[\[30\_Dibble \_image008.png]]

&#x20;

!\[\[30\_Dibble \_image009.png]]

&#x20;

From here, we have successfully identified a manner of which we can inject logs. Now, we just need to find out what kind of code we can inject from here. Perhaps we can try some SSTI, but I think RCE is possible from this vector.

&#x20;

We can confirm that 1+1 works here.

!\[\[30\_Dibble \_image0010.png]]

&#x20;

I knew this wasn't a PHP application, so it was probably Node.js. This wasn't an SSTI as it was purely executing code without any expression formation, so it has to be Javascript based.

&#x20;

Tried a few payloads for NodeJS from PayloadAllTheThings and this confirmed that it was indeed a Node.js application.

&#x20;

{width="4.875in" height="0.8541666666666666in"}

Tried this one for port 21.

!\[\[30\_Dibble \_image0012.png]]

And it worked!

!\[\[30\_Dibble \_image0013.png]]

Cool.

{width="8.291666666666666in" height="4.8125in"}

Right, time to PE.

&#x20;

However, there seems to be another issue whereby our connection is terminated really fast, perhaps due to a cronjob resetting connections or something.

&#x20;

Did a quick check on stuff using the find SUID binaries.

!\[\[30\_Dibble \_image0015.png]]

&#x20;

From here, we can clearly see that cp is enabled. This would mean we can copy files to the /etc/passwd folder with our own user with a root access.

&#x20;

{width="9.572916666666666in" height="2.0416666666666665in"}

&#x20;

{width="4.21875in" height="1.2395833333333333in"}

&#x20;

From here, we can grab the root flag.

&#x20;

{width="8.416666666666666in" height="4.6875in"}
