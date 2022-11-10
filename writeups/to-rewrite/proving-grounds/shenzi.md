# Shenzi

Shenzi

Friday, 18 March 2022

9:33 am

Another windows machine with these ports:

&#x20;

!\[\[92\_Shenzi\_image001.png]]

&#x20;

!\[\[92\_Shenzi\_image002.png]]

Port 80 and port 443 are just XAMPP servers.

&#x20;

Smbclient returns one drive:

!\[\[92\_Shenzi\_image003.png]]

&#x20;

There are some files within this:

!\[\[92\_Shenzi\_image004.png]]

&#x20;

I downloaded all the files there.

Passwords.txt contained some interesting information.

&#x20;

!\[\[92\_Shenzi\_image005.png]]

I see a wordpress and a webdav, and if webdav is enabled, then I can directly upload shells onto that web server.

&#x20;

&#x20;

!\[\[92\_Shenzi\_image006.png]]

Well this didn't work.

&#x20;

Well, off to find that wordpress website.

Searching /wordpress does not work, but appending shenzi to the end of the URL does let me find it

!\[\[92\_Shenzi\_image007.png]]

We could log in as the admin using those credentials.

&#x20;

!\[\[92\_Shenzi\_image008.png]]

&#x20;

From here, getting a shell is quite okay.

Go to appearance and click theme editor, then click edit 404.php and plant some php shell in there.

&#x20;

!\[\[92\_Shenzi\_image009.png]]

Almost worked!

&#x20;

We can just get a simple php cmd executor on this still:

!\[\[92\_Shenzi\_image0010.png]]

This works still.

&#x20;

Now from here, we can begin getting nc and stuff onto the machine, of which we already know that it is a 64bit machine.

Eventually, I get a shell on port 21, which is not blocked by the firewall.

!\[\[92\_Shenzi\_image0011.png]]

&#x20;

!\[\[92\_Shenzi\_image0012.png]]

&#x20;

!\[\[92\_Shenzi\_image0013.png]]

&#x20;

!\[\[92\_Shenzi\_image0014.png]]

&#x20;

This shell is a little unstable, and takes a while to execute code.

But it always eventually responds.

&#x20;

Downloaded winpeas onto this to run stuff for me.

Clearly, it was too much for it as my shell crashed :( Rebooting it is easy, thankfully.

&#x20;

Checking around, I found that AlwaysInstallElevated was enabled to 1.

!\[\[92\_Shenzi\_image0015.png]]

This would mean we can get some msi files within this.

Seems like there are a myriad of ports that are blocked...this took a long ass time.

&#x20;

Realised that the windows machine was on the verge of crashing due to the number of things I was downloading, and hence was not functioning properly.

&#x20;

Hence, I reset and tried again:

!\[\[92\_Shenzi\_image0016.png]]

&#x20;

This would almost work, so I tried once more.

&#x20;

!\[\[92\_Shenzi\_image0017.png]]

Great!

&#x20;

!\[\[92\_Shenzi\_image0018.png]]

&#x20;
