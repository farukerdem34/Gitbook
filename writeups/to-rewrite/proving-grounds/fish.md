# Fish

Fish

Tuesday, 15 March 2022

8:32 pm

My IP is 192.168.49.83.

The target IP is 192.168.73.168.

&#x20;

Interesting port numbers:

!\[\[83\_Fish\_image001.png]]

&#x20;

!\[\[83\_Fish\_image002.png]]

&#x20;

That's a lot of different port and stuff.

A quick google search about these services reveals that there are existing exploits to this version of GlassFish.

!\[\[83\_Fish\_image003.png]]

&#x20;

Anyways, let's view some of these ports, which all contain some form of web server with a file upload avenue except for port 4848, which contains some administrator panel.

&#x20;

!\[\[83\_Fish\_image004.png]]

My default credentials don't work here. Seems that the admin is not enabled on this?

&#x20;

Port 6060 contains another login page:

!\[\[83\_Fish\_image005.png]]

&#x20;

Ports 8080 and 8181 were not useful.

&#x20;

Anyways, looking at my earlier exploit, I could find one LFI for this Oracle GlassFish thing.

!\[\[83\_Fish\_image006.png]]

&#x20;

Also, this confirmed the version.

From here, I began enumerating the system:

!\[\[83\_Fish\_image007.png]]

&#x20;

I found this interesting file from /Synaman/config/afumap.dat.

&#x20;

!\[\[83\_Fish\_image008.png]]

&#x20;

Within the AppConfig.xml, I found another password.

!\[\[83\_Fish\_image009.png]]

Some more passwords, and another user.

&#x20;

From this, we can RDP in, because port 3389 using those credentials.

!\[\[83\_Fish\_image0010.png]]

&#x20;

We can actually grab the local flag from this.

!\[\[83\_Fish\_image0011.png]]

&#x20;

Best part about this is that I can actually just download winpeas onto this, albeit without the nice colours.

&#x20;

!\[\[83\_Fish\_image0012.png]]

No interesting permissions for us.

&#x20;

!\[\[83\_Fish\_image0013.png]]

Plenty of hotfixes that do things we cannot bypass, and hence there are no vulnerabilities recommended by watson.

&#x20;

I couldn't take this shell anymore, so I switched by downloading netcat onto this and then doing a shell to my machine.

!\[\[83\_Fish\_image0014.png]]

!\[\[83\_Fish\_image0015.png]]

&#x20;

Ran a winpeas again, this time in colour.

{width="8.5625in" height="1.9375in"}

&#x20;

While on the rdp client, I kept seeing this popup.

!\[\[83\_Fish\_image0017.png]]

Looking into total AV, I saw this one exploit.

&#x20;

!\[\[83\_Fish\_image0018.png]]

&#x20;

!\[\[83\_Fish\_image0019.png]]

Seems interesting, and I wanted to try it out. Normally, why would a Windows machine contain an AV (that clearly isn't blocking my downloads or anything lol).

&#x20;

Followed the video exactly, first create a malicious dll file.

{width="9.760416666666666in" height="2.0in"}

Transfer it over to the machine using powershell -c wget.

&#x20;

From here, we need to use RDP.

Move the dll to any file. I moved mine to a new file called mount in C:\mount

From here, boot up the AV, and head to quick scan > private scan.

&#x20;

!\[\[83\_Fish\_image0021.png]]

Finish the scan and ensure it has been identified as a threat, then choose remove threats and then and choose quarantine!

&#x20;

!\[\[83\_Fish\_image0022.png]]

&#x20;

!\[\[83\_Fish\_image0023.png]]

&#x20;

Start an exploit handler:

!\[\[83\_Fish\_image0024.png]]

&#x20;

Create a symbolic link.

&#x20;

From here, ensure that the file is in quarantine and now gone from the usual directory.

!\[\[83\_Fish\_image0025.png]]

&#x20;

Delete the mount folder and then create a new symlink.

!\[\[83\_Fish\_image0026.png]]

&#x20;

To verify it worked, check the Framework directory and verify that there is indeed the version.dll within it.

!\[\[83\_Fish\_image0027.png]]

&#x20;

Then restart the machine.

We will get a shell!

{width="6.65625in" height="3.875in"}

&#x20;

!\[\[83\_Fish\_image0029.png]]

Pwned. Took long because I'm an idiot.

&#x20;

&#x20;
