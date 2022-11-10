# CookieCutter

CookieCutter

Thursday, 24 March 2022

10:14 pm

This Linux machine has these ports.

&#x20;

Interestingly, there's a little tidbit on the page source for port 80.

&#x20;

!\[\[39\_CookieCutter\_image001.png]]

&#x20;

When we download this script, we can see that this would download a script with functions that send commands to an external server. Perhaps this is a buffer overflow of some sort.

{width="3.9375in" height="7.614583333333333in"}

&#x20;

There seem to be two forms of functions, the start being the login of some sort. The second function is called through the use of another function.

So we know the user is bob, but they hid the password.

&#x20;

Interestingly, there is also another command section here, whereby we do not know what it can do.

{width="6.770833333333333in" height="5.208333333333333in"}

&#x20;

So we know these functions are already enabled. For this function, it seems to take the command as the second parameter after a null byte. This would indicate that the kind of commands that we can enter.

&#x20;

First order of business, we need to brute force this password until we get it. For now, we can include a print(data) to determine if we have logged in correctly or not.

!\[\[39\_CookieCutter\_image004.png]]

We can create this custom script to run for us.

This would brute force the password and hopefully find it.

&#x20;

&#x20;

{width="3.9791666666666665in" height="2.9791666666666665in"}

&#x20;

Eventually after a long time, it found it!

Takes a while, but eventually it finds it.

&#x20;

{width="6.0in" height="0.71875in"}

This should be the password. Great! Now, let's move on to the command portion.

That center portion should be the UUID, which would be the part we send to the next function.

&#x20;

{width="5.583333333333333in" height="1.40625in"}

This seems to be a wrong command.

&#x20;

For now, we need to think of commands that can be executed on the server side.

!\[\[39\_CookieCutter\_image008.png]]

&#x20;

Let's just try sending this one.

{width="5.927083333333333in" height="1.375in"}

&#x20;

Interesting, there is a curl command here!

!\[\[39\_CookieCutter\_image0010.png]]

This is who we are, and we can curl ourselves. This could be used to execute shells perhaps.

&#x20;

This box is real challenging, and I'm glad I figured out the first portion.

&#x20;

Anyways from here, I did the rest in office.

&#x20;

We need to do some SSRF and curl 127.0.0.1, which would reveal some information on both ports 80 and 8080.

&#x20;

On port 8080, there is this one part whereby it echos a string.

!\[\[39\_CookieCutter\_image0011.png]]

From here, we can do some SSTI to get an RCE to gain a reverse shell with the parameter of echostr appended to the end of the query.

&#x20;

!\[\[39\_CookieCutter\_image0012.png]]

&#x20;

In this case, the offset is 400 for it. From here, we can continue to gain shell on the machine and grab the user flag.

!\[\[39\_CookieCutter\_image0013.png]]

&#x20;

From here within the code, we can try to find the password.

!\[\[39\_CookieCutter\_image0014.png]]

&#x20;

There is the source code within the directory of bob the user.

&#x20;

From there we can just read the source code, and compile it on our computer, appending the printf(password) function there so we don't need to guess it.

!\[\[39\_CookieCutter\_image0015.png]]

After getting the password, we can check sudo rights.

&#x20;

!\[\[39\_CookieCutter\_image0016.png]]

This is definitely some custom stuff, and what we can do is this for a root shell.

&#x20;

!\[\[39\_CookieCutter\_image0017.png]]

&#x20;

This works because of the privileges that the python command prompt can do:

!\[\[39\_CookieCutter\_image0018.png]]

&#x20;

!\[\[39\_CookieCutter\_image0019.png]]

&#x20;

Take note that I took a walkthrough for this after a while because I was getting too stuck and did not know what to do. Cheers to this challenging yet fun box.

&#x20;

&#x20;
