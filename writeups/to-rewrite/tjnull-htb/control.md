# Control

Control

Monday, 14 March 2022

6:08 pm

This is a Windows machine.

The target IP is 10.10.10.167.

My IP is 10.10.16.9.

&#x20;

Enumerating this, there seems to be the usual ports.

&#x20;

!\[\[86\_Control\_image001.png]]

Right, so from here, we need to enumerate the HTTP and MSRPC stuff.

&#x20;

!\[\[86\_Control\_image002.png]]

&#x20;

HTTP:

!\[\[86\_Control\_image003.png]]

Right off the bat, I tried to access the Login page, but there was an error.

&#x20;

!\[\[86\_Control\_image004.png]]

Hmm..

Did a directory enumeration and found out some interesting directories:

!\[\[86\_Control\_image005.png]]

Database.php is one to look for, and there's also an uploads/ directory.

&#x20;

Anyways, the header means that we need to use some HTTP headers to spoof the proxy. In this case, X-Forwarded-For came to mind because of the PortSwigger Academy.

&#x20;

Somehow, we should be able to include this and browse the website.

&#x20;

Looking at the web page source, we can see this:

!\[\[86\_Control\_image006.png]]

This looks to be an IP address that works, let's try it.

&#x20;

!\[\[86\_Control\_image007.png]]

Aight, this works.

&#x20;

We just need to append this to the requests that we do.

There should be a tool which we can use to modify the HTTP headers, in which case there are a lot of extensions that do just that.

&#x20;

!\[\[86\_Control\_image008.png]]

&#x20;

&#x20;

Afterwards, we can access the page:

!\[\[86\_Control\_image009.png]]

From here I saw a search bar, and just had to try SQL Injection, which worked!

&#x20;

{width="14.8125in" height="1.1875in"}

&#x20;

From there, we know that this is using MariaDB, which I have some stuff for.

I used sqlmap just to prove this, but I did not want to exploit the server using SQL Map.

&#x20;

&#x20;

Wanted to do myself.

&#x20;

From here, I first tried some UNION injection to determine the columns.

!\[\[86\_Control\_image0011.png]]

&#x20;

{width="8.041666666666666in" height="0.84375in"}

If there's an error, it means I got the wrong stuff.

&#x20;

Seems that 6 NULLs give me no errors, which means there are 6 columns available.

Looking around the web, there is a method to include files that are to be written within the folders.

!\[\[86\_Control\_image0013.png]]

&#x20;

!\[\[86\_Control\_image0014.png]]

&#x20;

Knowing that there are 6 columns, I would perhaps write this file within the directory somehow.

!\[\[86\_Control\_image0015.png]]

&#x20;

Great, now we should have some form of RCE.

!\[\[86\_Control\_image0016.png]]

Realised we should use this.

&#x20;

Now, we can test this theory.

!\[\[86\_Control\_image0017.png]]

Hmm...

&#x20;

Changed the command string to this:

!\[\[86\_Control\_image0018.png]]

Great, now it works.

!\[\[86\_Control\_image0019.png]]

&#x20;

Now let's upload some netcat.

!\[\[86\_Control\_image0020.png]]

&#x20;

!\[\[86\_Control\_image0021.png]]

Alright.

&#x20;

From here, we can view the database.php file.

&#x20;

!\[\[86\_Control\_image0022.png]]

There is one other user on this machine:

&#x20;

!\[\[86\_Control\_image0023.png]]

&#x20;

Hector is part of the Remote Management Use system!

&#x20;

!\[\[86\_Control\_image0024.png]]

This means if I can find credentials for hector, I can use evil-winrm on him.

&#x20;

Within the MariaDB directory, I found the global\_priv.MAD file which has the passwords for each user encrypted with a hash.

&#x20;

!\[\[86\_Control\_image0025.png]]

&#x20;

I think this was meant to be accessed through SQL Injection, but whatever.

&#x20;

!\[\[86\_Control\_image0026.png]]

This is the hash for hector.

&#x20;

When decrypted, it gives us his password.

&#x20;

!\[\[86\_Control\_image0027.png]]

Evil-winrm does not work, so let's do it the other method using powershell, and scriptblocks!

!\[\[86\_Control\_image0028.png]]

&#x20;

!\[\[86\_Control\_image0029.png]]

Right, so this works!

&#x20;

Now, we just need to activate the reverse shell again.

!\[\[86\_Control\_image0030.png]]

&#x20;

!\[\[86\_Control\_image0031.png]]

Great.

&#x20;

Now from here, we can continue.

&#x20;

Hector has basically no privileges.

!\[\[86\_Control\_image0032.png]]

I can't even run a systeminfo using this.

&#x20;

Ran a winpeas on this instead. Lazy...

&#x20;

Interestingly, Hector has some powershell history.

&#x20;

!\[\[86\_Control\_image0033.png]]

&#x20;

!\[\[86\_Control\_image0034.png]]

This could be a hint.

&#x20;

So firstly, we notice that this whole directory is owned by the administrator.

!\[\[86\_Control\_image0035.png]]

&#x20;

From here, we can identify certain services that are to be used when we are doing stuff.

&#x20;

Reading the ACL, there are ways to convert it to readable text:

!\[\[86\_Control\_image0036.png]]

&#x20;

Let's view hector's privileges.

!\[\[86\_Control\_image0037.png]]

Looks like we have loads of privileges over these registries.

&#x20;

While googling about tools to use, I found this one:

!\[\[86\_Control\_image0038.png]]

&#x20;

When viewing the output, this popped up, from [this](https://www.hackingarticles.in/windows-privilege-escalation-weak-registry-permission/) blog.

I downloaded accesschk.exe and uploaded it to the machine and ran it as per the blog.

{width="4.75in" height="0.8229166666666666in"}

&#x20;

This was for this directory:

!\[\[86\_Control\_image0040.png]]

&#x20;

Looking more at winpeas, it seems that we can modify lots of services.

&#x20;

!\[\[86\_Control\_image0041.png]]

Through lots of searching, we had to find a service that runs on the local system.

&#x20;

!\[\[86\_Control\_image0042.png]]

Then, we need to find the one that has our privleges.

!\[\[86\_Control\_image0032.png]]

&#x20;

Find one that uses these privileges, or nothing at all even better.

Afterwards, these services need to have a start value of 3.

&#x20;

!\[\[86\_Control\_image0043.png]]

This means we need to start the service ourselves.

&#x20;

From there, we need to be able to write the image path to this. Afterwards, we can change the image path in order to change the execution of the file to the nc path that we can use gain a reverse shell.

&#x20;

Doing so, I think that this service works.

!\[\[86\_Control\_image0044.png]]

&#x20;

Hector also is able to control this one.

!\[\[86\_Control\_image0045.png]]

&#x20;

{width="7.833333333333333in" height="3.15625in"}

This one seems to work.

&#x20;

From there, we can just alter the bin path and get it to execute the nc back to us again.

Well, that didn't work.

&#x20;

{width="14.010416666666666in" height="1.4270833333333333in"}

Not that easy...

&#x20;

Well as it turns out, I can also use this command to alter the services:

!\[\[86\_Control\_image0048.png]]

This works in giving me a shell.

&#x20;

!\[\[86\_Control\_image0049.png]]

&#x20;

And this shell is the admin.

&#x20;

!\[\[86\_Control\_image0050.png]]

Great.

&#x20;

Quite happy, didn't use SQLMap.

&#x20;

&#x20;

&#x20;
