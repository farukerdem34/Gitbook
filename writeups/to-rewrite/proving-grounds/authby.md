# AuthBy

AuthBy

Wednesday, 16 March 2022

1:09 pm

My IP is 192.168.49.61.

The target IP is 192.168.61.46.

&#x20;

!\[\[85\_AuthBy\_image001.png]]

&#x20;

!\[\[85\_AuthBy\_image002.png]]

Another box here, with some interesting ports.

&#x20;

When I see FTP, I try anonymous logins, which works.

!\[\[85\_AuthBy\_image003.png]]

Nothing much there though.

&#x20;

From here, we can determine some account names.

&#x20;

{width="7.958333333333333in" height="1.4270833333333333in"}

&#x20;

When trying FTP again, we can login as admin:admin.

When viewing the code, we can see there's some hidden files showed to us:

!\[\[85\_AuthBy\_image005.png]]

&#x20;

I grabbed all of these files.

&#x20;

From here, we can grab a hash from that htpasswd file.

!\[\[85\_AuthBy\_image006.png]]

&#x20;

Running a john, we can crack this:

{width="10.125in" height="2.7604166666666665in"}

Now, we have some credentials.

These do not work on the FTP server.

&#x20;

Let's look at the HTTP website.

When accessing port 242, we need to provide credentials:

!\[\[85\_AuthBy\_image008.png]]

The credentials we found works, and we are greeted with this:

!\[\[85\_AuthBy\_image009.png]]

&#x20;

!\[\[85\_AuthBy\_image0010.png]]

Right.

This was a website, and the earlier file had a .htpasswd file, and indicated index.php

!\[\[85\_AuthBy\_image0011.png]]

This confirmed I had access to the web root, and could perhaps upload some reverse shells within the machine.

&#x20;

Log in to FTP and then put the file in there.

&#x20;

!\[\[85\_AuthBy\_image0012.png]]

&#x20;

Execution through the web browser works, but I need to find a better reverse shell.

&#x20;

!\[\[85\_AuthBy\_image0013.png]]

Uploaded my one liner php shell, and we have RCE!

!\[\[85\_AuthBy\_image0014.png]]

Cool.

Now we have RCE, we can view some stuff.

Well, downloading Netcat does not work, so let's try other methods.

&#x20;

I created a reverse.exe file from MSFvenom.

{width="10.010416666666666in" height="1.8333333333333333in"}

Then I placed it within the web directory.

!\[\[85\_AuthBy\_image0016.png]]

&#x20;

Ran it using my cmd.php.

!\[\[85\_AuthBy\_image0017.png]]

&#x20;

And we would get a shell.

&#x20;

!\[\[85\_AuthBy\_image0018.png]]

&#x20;

!\[\[85\_AuthBy\_image0019.png]]

&#x20;

I got winpeas on the machine, and enumerated a bit before that.

!\[\[85\_AuthBy\_image0020.png]]

There's the impersonate privilege thing enabled, and when looking at the machine itself, it had no hotfixes.

{width="9.34375in" height="7.302083333333333in"}

&#x20;

I downloaded the winPEAS through certutil.

!\[\[85\_AuthBy\_image0022.png]]

Well, it does not let me run it, and powershell is oddly not on this machine.

&#x20;

Interesting.

&#x20;

Anyways, from this, I already knew that we had to use JuicyPotato, but the x86 version. This was because the impersonateprivileges was enabled, there seem to be no hotfixes, which indicates that it there are no patches for vulnerabilities.

&#x20;

From here, we also need to get nc32.exe

!\[\[85\_AuthBy\_image0023.png]]

WE should have these within any directory.

&#x20;

!\[\[85\_AuthBy\_image0024.png]]

Run this and on our listener port...

!\[\[85\_AuthBy\_image0025.png]]

&#x20;

!\[\[85\_AuthBy\_image0026.png]]

Weird machine does not have powershell, but to be fair we had certutil and smb server to use incase.

&#x20;

&#x20;
