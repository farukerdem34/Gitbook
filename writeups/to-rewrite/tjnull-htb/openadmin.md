# OpenAdmin

OpenAdmin

Saturday, 12 February 2022

9:55 pm

This is a Linux machine.

My IP is 10.10.16.9.

The target IP is 10.10.10.171.

&#x20;

!\[\[34\_OpenAdmin\_image001.png]]

&#x20;

SSH and HTTP.

!\[\[34\_OpenAdmin\_image002.png]]

&#x20;

Default page, hence we should run both directory and vhost stuff in case.

!\[\[34\_OpenAdmin\_image003.png]]

&#x20;

From here, I checked each website out. To save the time, there's a link in the music one that brought me to a version of OpenNetAdmin.

&#x20;

!\[\[34\_OpenAdmin\_image004.png]]

Searchsploit reveals there is indeed an RCE for it for that exact version.

!\[\[34\_OpenAdmin\_image005.png]]

Checked it out, has a command to use!

!\[\[34\_OpenAdmin\_image006.png]]

&#x20;

So to test it out, I just removed the final parts of sed and tail, and appended my command I wanted to do within the code there.

!\[\[34\_OpenAdmin\_image007.png]]

&#x20;

Did an ID to check.

&#x20;

!\[\[34\_OpenAdmin\_image008.png]]

&#x20;

And it responds to me well. From here, we just need to inject a reverse shell and we should be good. Be sure to URL encode some parts, as curl would not read it properly.

&#x20;

!\[\[34\_OpenAdmin\_image009.png]]

&#x20;

!\[\[34\_OpenAdmin\_image0010.png]]

Within the file we spawned in, there's a database password.

&#x20;

!\[\[34\_OpenAdmin\_image0011.png]]

There are two users within the machine as well.

&#x20;

!\[\[34\_OpenAdmin\_image0012.png]]

We can't access either of them as root.

&#x20;

I tried to SSH into both accounts, and the user jimmy worked with this password.

&#x20;

!\[\[34\_OpenAdmin\_image0013.png]]

However, there was no user flag within this account, so we need to keep looking.

&#x20;

When I chanced upon the /var/www/internal directory, I found some interesting files.

&#x20;

&#x20;

!\[\[34\_OpenAdmin\_image0014.png]]

&#x20;

{width="15.28125in" height="1.3020833333333333in"}

&#x20;

We need to find this internal website, and I know it's on an internal Apache server.

&#x20;

Go to /etc/apache2/sites-available directory to find that hidden site.

&#x20;

!\[\[34\_OpenAdmin\_image0016.png]]

&#x20;

Do a quick SSH tunnel.

&#x20;

!\[\[34\_OpenAdmin\_image0017.png]]

After this we can access the site.

&#x20;

&#x20;

!\[\[34\_OpenAdmin\_image0018.png]]

&#x20;

!\[\[34\_OpenAdmin\_image0019.png]]

From here, decrypt the hash.

&#x20;

Use that to login and get the id\_Rsa key.

&#x20;

!\[\[34\_OpenAdmin\_image0020.png]]

&#x20;

Decrypt using openssl, however the ninja keyword does not work.

I used rockyou.txt, and eventually found it.

SSh2john it first though.

&#x20;

{width="10.635416666666666in" height="2.4375in"}

Openssl decrypt it with bloodninjas as the key.

&#x20;

!\[\[34\_OpenAdmin\_image0022.png]]

Use that key to log in!

!\[\[34\_OpenAdmin\_image0023.png]]

Sudo -l to find out what I can do.

&#x20;

!\[\[34\_OpenAdmin\_image0024.png]]

How nano works is basically it can read or write to a file.

&#x20;

!\[\[34\_OpenAdmin\_image0025.png]]

&#x20;

We have one file to work with, and we can make it work.

&#x20;

&#x20;

!\[\[34\_OpenAdmin\_image0026.png]]

&#x20;

!\[\[34\_OpenAdmin\_image0027.png]]

Basically, we just do

Nano -s /opt/priv

Then ^R^x and that command.

&#x20;

This would spawn us a shell within nano.

&#x20;

!\[\[34\_OpenAdmin\_image0028.png]]

Pwned.

&#x20;

&#x20;
