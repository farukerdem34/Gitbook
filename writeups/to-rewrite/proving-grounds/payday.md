# Payday

Payday

Tuesday, 15 March 2022

5:37 pm

The target IP is 192.168.73.39.

My IP is 192.168.49.73.

&#x20;

!\[\[04\_Payday\_image001.png]]

Alright.

!\[\[04\_Payday\_image002.png]]

&#x20;

Web page show this really outdated internet shop.

&#x20;

!\[\[04\_Payday\_image003.png]]

&#x20;

There's a login at the side.

&#x20;

Let's searchsploit it.

!\[\[04\_Payday\_image004.png]]

Alright, lots of different types of exploits.

&#x20;

I tried default credentials of admin:admin and logged in.

&#x20;

!\[\[04\_Payday\_image005.png]]

&#x20;

!\[\[04\_Payday\_image006.png]]

&#x20;

Anyways, while looking at exploits, I determined that there was an LFI and confirmed the version number.

&#x20;

!\[\[04\_Payday\_image007.png]]

&#x20;

!\[\[04\_Payday\_image008.png]]

Right, also I found an RCE for this version, on Github.

&#x20;

!\[\[04\_Payday\_image009.png]]

&#x20;

Visit admin.php.

!\[\[04\_Payday\_image0010.png]]

Login with admin:admin.

&#x20;

From there, we can follow the exploit.

!\[\[04\_Payday\_image0011.png]]

&#x20;

!\[\[04\_Payday\_image0012.png]]

&#x20;

!\[\[04\_Payday\_image0013.png]]

&#x20;

!\[\[04\_Payday\_image0014.png]]

&#x20;

!\[\[04\_Payday\_image0015.png]]

And we're in. Upgrade the shell using TTY and stty raw -echo;fg.

&#x20;

We can access the user patrick's home directory and capture our flag.

&#x20;

!\[\[04\_Payday\_image0016.png]]

&#x20;

Within the webroot, we can see a db file there.

&#x20;

!\[\[04\_Payday\_image0017.png]]

There's nothing in there though.

&#x20;

I ran a linpeas on this.

&#x20;

!\[\[04\_Payday\_image0018.png]]

&#x20;

!\[\[04\_Payday\_image0019.png]]

&#x20;

Within the root directory, there was this one user allowed to SSH in, but I can't append my own key to that file.

&#x20;

!\[\[04\_Payday\_image0020.png]]

&#x20;

Do we need to pivot elsewhere?

&#x20;

Then I saw this:

!\[\[04\_Payday\_image0021.png]]

Well...weird configurations for the .ssh isn't it.

\\

!\[\[04\_Payday\_image0022.png]]

Okay...

&#x20;

So we can login to mysql from here.

!\[\[04\_Payday\_image0023.png]]

&#x20;

!\[\[04\_Payday\_image0024.png]]

D

!\[\[04\_Payday\_image0025.png]]

&#x20;

&#x20;

!\[\[04\_Payday\_image0026.png]]

Nothing much there though.

!\[\[04\_Payday\_image0027.png]]

&#x20;

!\[\[04\_Payday\_image0028.png]]

The admin and customer passwords are admin and customer. The root password for MySQL was root.

&#x20;

Well, we know there's a user called patrick, why not try an su there?

!\[\[04\_Payday\_image0029.png]]

The password was patrick.

&#x20;

{width="7.0in" height="2.9166666666666665in"}

Well...

!\[\[04\_Payday\_image0031.png]]

&#x20;

!\[\[04\_Payday\_image0032.png]]

For a few seconds I was afraid I had to pivot to the other explorer pc and then SSH in from there. Thank god.

&#x20;
