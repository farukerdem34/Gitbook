# BackendTwo

BackendTwo

Thursday, June 23, 2022

9:50 AM

Nmap scan:

!\[\[32\_BackendTwo\_image001.png]]

&#x20;

This is largely the same box as BackEnd one, which os on the page before this one.

Think of it like an updated version of the one before.

&#x20;

Port 80:

!\[\[32\_BackendTwo\_image002.png]]

&#x20;

We can view the administrator details:

!\[\[32\_BackendTwo\_image003.png]]

&#x20;

!\[\[32\_BackendTwo\_image004.png]]

&#x20;

We can do the signup again.

!\[\[32\_BackendTwo\_image005.png]]

&#x20;

But first, this one seems to block most emails that I have tried, except this one:

!\[\[32\_BackendTwo\_image006.png]]

&#x20;

Then we can login to receive another token:

!\[\[32\_BackendTwo\_image007.png]]

&#x20;

Using this token, we can access the docs again.

!\[\[32\_BackendTwo\_image008.png]]

&#x20;

There was one new endpoint, which was to edit a user's profile:

!\[\[32\_BackendTwo\_image009.png]]

&#x20;

In our case, our JWT Token tells us that our id is 12.

!\[\[32\_BackendTwo\_image0010.png]]

&#x20;

01 is the administrator as usual.

So this one takes a JSON value with an integer, which I presume is the profile number.

!\[\[32\_BackendTwo\_image0011.png]]

&#x20;

This just returns true.

!\[\[32\_BackendTwo\_image0012.png]]

&#x20;

I'm assuming I can do stuff like change my password or something.

&#x20;

!\[\[32\_BackendTwo\_image0013.png]]

&#x20;

I accidentally submitted my profile as the integer 1, which returns something interesting.

!\[\[32\_BackendTwo\_image0014.png]]

&#x20;

Seems that my profile has changed.

If I try to edit the administrator's account with id=1, I get this error.

!\[\[32\_BackendTwo\_image0015.png]]

&#x20;

So I'm not allowed to change stuff like that.

&#x20;

Editing username:

!\[\[32\_BackendTwo\_image0016.png]]

&#x20;

!\[\[32\_BackendTwo\_image0017.png]]

&#x20;

I find out that I'm allowed to edit my email. So in this case, I wondered if I could change my guid to the administrator's or something.

And I certainly can!

!\[\[32\_BackendTwo\_image0018.png]]

&#x20;

In fact, I can totally take over the administrator account.

I can make myself a superuser.

!\[\[32\_BackendTwo\_image0019.png]]

&#x20;

!\[\[32\_BackendTwo\_image0020.png]]

&#x20;

And with this, I can now login to the admin panel after relogging and getting a new token.

!\[\[32\_BackendTwo\_image0021.png]]

&#x20;

Flag:

!\[\[32\_BackendTwo\_image0022.png]]

&#x20;

Now, reviewing the docs, I can do different things with this administrator account.

&#x20;

Get File:

!\[\[32\_BackendTwo\_image0023.png]]

&#x20;

Write File:

!\[\[32\_BackendTwo\_image0024.png]]

&#x20;

Interesting, because I am now allowed to get files instead of execute commands.

We just need to have the file name that is base64 encoded.

&#x20;

Encoding /etc/passwd:

!\[\[32\_BackendTwo\_image0025.png]]

&#x20;

!\[\[32\_BackendTwo\_image0026.png]]

&#x20;

We can now get this file. Interestingly, we are allowed to edit files. Again, we can view the /proc/self/environ file to see what is the home directory of this website.

!\[\[32\_BackendTwo\_image0027.png]]

&#x20;

So we know that the home directory of the user is htb. However, the same directory does not work in this case.

We can use our machine to guess what we wanna view, and try to find out where the web server is running from. I'm guessing we need to edit the app.py to add another command or endpoint to let us achieve RCE.

&#x20;

I just tried /home/htb/app/main.py and found it.

!\[\[32\_BackendTwo\_image0028.png]]

&#x20;

We can go down this hole again and this time, try to find out the secret of which the app runs on.

!\[\[32\_BackendTwo\_image0029.png]]

&#x20;

We can see that the main.py imports stuff from here, so let's take a look at it.

!\[\[32\_BackendTwo\_image0030.png]]

&#x20;

Within the config.py file, we can see that the JWT secret is hidden in the environ file, of which I did not see it earlier.

!\[\[32\_BackendTwo\_image0031.png]]

68b329da9893e34099c7d8ad5cb9c940

This is the key from the environ file.

&#x20;

So now, we can successfully make a valid token.

With this, we can now write files.

!\[\[32\_BackendTwo\_image0032.png]]

&#x20;

!\[\[32\_BackendTwo\_image0033.png]]

&#x20;

This was written to /tmp/test.txt.

&#x20;

Now, we can try to append code within the python script that would let me gain a reverse shell.

Now all we had to do was set up some condition within one of the python scripts to have a python reverse shell, and have a unique condition such that only I can trigger it.

&#x20;

I took a look at the endpoint user.py to see what it does, and this seems like a good fit.

Original function:

!\[\[32\_BackendTwo\_image0034.png]]

&#x20;

Edited function:

!\[\[32\_BackendTwo\_image0035.png]]

&#x20;

Now, we have a condition that we can use, we just need to find a way to curl this...somehow...

Fortunately, cyberchef has an escape string function that can do that for us.

&#x20;

!\[\[32\_BackendTwo\_image0036.png]]

&#x20;

Just set to escape all double quotes.

Then, we can try to curl this in somehow.

&#x20;

We end up with something like this:

!\[\[32\_BackendTwo\_image0037.png]]

&#x20;

!\[\[32\_BackendTwo\_image0038.png]]

&#x20;

Then we just need to visit the fetch user id page with an id of -100.

!\[\[32\_BackendTwo\_image0039.png]]

&#x20;

{width="8.03125in" height="2.59375in"}

&#x20;

PE:

When we try to sudo...

{width="6.1875in" height="3.4791666666666665in"}

&#x20;

I have no idea what this is ngl lol.

Auth.log:

!\[\[32\_BackendTwo\_image0042.png]]

&#x20;

This isn't the root password, but rather the password for htb user.

I would play the wordle, but it doesn't accept any guesses I do.

&#x20;

In this case, we should go find out where this binary is and perhaps RE it.

Checking the shared libraries and stuff in the /etc/pam.d file, we can see that there's this pam\_world.so file.

!\[\[32\_BackendTwo\_image0043.png]]

&#x20;

!\[\[32\_BackendTwo\_image0044.png]]

&#x20;

Doing strings on this results in there being the directory given here.

!\[\[32\_BackendTwo\_image0045.png]]

&#x20;

After playing with this and grep, I managed to sudo /bin/bash.

{width="4.90625in" height="4.395833333333333in"}

&#x20;

Flag:

!\[\[32\_BackendTwo\_image0047.png]]

8b68e76bcdc2aa9f1d844081344268b7

&#x20;
