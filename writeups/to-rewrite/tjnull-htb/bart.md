# Bart

Bart

Monday, 14 March 2022

4:11 pm

This is a Windows machine.

The target IP is 10.10.10.81.

My IP is 10.10.16.9.

&#x20;

!\[\[85\_Bart\_image001.png]]

Pure web it seems.

&#x20;

Website:

!\[\[85\_Bart\_image002.png]]

This is some kind of forum.

&#x20;

Looking around, there's one team to take note of:

!\[\[85\_Bart\_image003.png]]

Right.

&#x20;

When viewing the page source, we can see this:

&#x20;

!\[\[85\_Bart\_image004.png]]

Ran a vuln scan on this, which returned some interesting things.

&#x20;

!\[\[85\_Bart\_image005.png]]

&#x20;

Ran a quick vhost enumeration too, as directory enumeration was not working out for me.

&#x20;

!\[\[85\_Bart\_image006.png]]

Right, add that to the hosts file.

&#x20;

From here, we have a log in page.

!\[\[85\_Bart\_image007.png]]

Some version down there too.

&#x20;

Now, we can use the forgot password and key in a username, and it seems to verify which users are existing.

&#x20;

!\[\[85\_Bart\_image008.png]]

The username harvey worked, and since the developer that was commented out had a name of harvey potter, I tried potter as the password.

This worked out...

!\[\[85\_Bart\_image009.png]]

&#x20;

!\[\[85\_Bart\_image0010.png]]

Click that internal chat thing, and we are able to view information about another web-server.

&#x20;

!\[\[85\_Bart\_image0011.png]]

Now there was another login page here, I tried the same credentials of harvey and potter.

{width="10.822916666666666in" height="5.3125in"}

This did not work, as it said that the password must be more than 8 characters.

&#x20;

{width="4.28125in" height="3.9791666666666665in"}

However, a different message is given when we give a false username.

&#x20;

{width="4.177083333333333in" height="3.78125in"}

&#x20;

{width="3.9791666666666665in" height="3.75in"}

Now, at least we know that harvey is a good password.

I decided to run a directory

!\[\[85\_Bart\_image0016.png]]

&#x20;

Seems that there is some register.php. Doesn't seem to accept anything though.

Hmm, but it is there, just that it always returns code 500 for some reason, and perhaps I can create a post request to just register anyways.

I looked to github to find a simple chat PHP application, which returned something cool.

&#x20;

!\[\[85\_Bart\_image0017.png]]

The repo looks similar to our dir enumeration.

&#x20;

!\[\[85\_Bart\_image0018.png]]

Looking at the code for the register.php, it seems to take two parameters, which are uname and passwd respectively.

&#x20;

{width="5.0in" height="3.5416666666666665in"}

So we just need to craft a quick POST request to send these to the host.

&#x20;

We can then register and login.

&#x20;

!\[\[85\_Bart\_image0020.png]]

This seems to work as we are redirected to the login page.

&#x20;

We can then login using these credentials and view the chat stuff.

&#x20;

!\[\[85\_Bart\_image0021.png]]

When we click the log, we just get a popup with a 1 in it.

!\[\[85\_Bart\_image0022.png]]

&#x20;

Viewing the request, it seems to take a filename parameter.

&#x20;

!\[\[85\_Bart\_image0023.png]]

Perhaps directory traversal works.

&#x20;

Definitely does, as when we play with it, we get this:

!\[\[85\_Bart\_image0024.png]]

This is an insecure function, and now we know the directory we are in.

&#x20;

We can actually view the logs from this file:

!\[\[85\_Bart\_image0025.png]]

&#x20;

Now, I wanted to see if we could vary some parameters here, because all it shows is the User-Agent.

!\[\[85\_Bart\_image0026.png]]

&#x20;

{width="2.25in" height="0.6875in"}

This works, we are able to control the User-Agent field.

&#x20;

Googling, it seems that we are able to get the browser to execute PHP c ode within this.

{width="7.375in" height="3.2916666666666665in"}

This works apparently.

&#x20;

!\[\[85\_Bart\_image0029.png]]

Eventually, I realise we need to change the log.txt to log.php in order to execute our code. Afterwards, this should work.

!\[\[85\_Bart\_image0030.png]]

We now have RCE.

&#x20;

Just need to get nc onto the machine.

!\[\[85\_Bart\_image0031.png]]

Simple enough, this would install, and now we just run the application.

&#x20;

This didn't work however, even though NC was gotten successfully. Got to think of other methods.

&#x20;

Decided to use some powershell to get it in there instead, through Invoke-PowerShellTcp.ps1

&#x20;

!\[\[85\_Bart\_image0032.png]]

&#x20;

!\[\[85\_Bart\_image0033.png]]

Cool.

&#x20;

!\[\[85\_Bart\_image0034.png]]

We are this guy.

Looking at our privileges, we have one interesting one:

&#x20;

!\[\[85\_Bart\_image0035.png]]

&#x20;

{width="6.802083333333333in" height="7.989583333333333in"}

No hotfixes either.

&#x20;

This version is vulnerable to JuicyPotato indeed.

Create some nc.bat file with these:

!\[\[85\_Bart\_image0037.png]]

&#x20;

!\[\[85\_Bart\_image0038.png]]

Set up a listener port.

From here, we need to find some CSIDs, and we can do so [here](http://ohpe.it/juicy-potato/CLSID/Windows\_10\_Pro/). Just pick anyone we want.

&#x20;

From there, we can just execute this command and it would test the CLSID and also determine a port to listen on and all that.

&#x20;

{width="10.104166666666666in" height="1.84375in"}

&#x20;

!\[\[85\_Bart\_image0040.png]]

Right.

!\[\[85\_Bart\_image0041.png]]

Because we picked the authority, we instantly become root.

So from there, we can grab both the flags at once.

&#x20;

&#x20;

&#x20;
