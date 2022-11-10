# Backend

Backend

Thursday, June 23, 2022

8:44 AM

Nmap scan:

!\[\[31\_Backend\_image001.png]]

&#x20;

Port 80:

!\[\[31\_Backend\_image002.png]]

&#x20;

Cool.

!\[\[31\_Backend\_image003.png]]

&#x20;

!\[\[31\_Backend\_image004.png]]

&#x20;

!\[\[31\_Backend\_image005.png]]

&#x20;

So there's a user and an admin.

When fuzzing this, we get some unique responses:

!\[\[31\_Backend\_image006.png]]

&#x20;

Seems that this directory takes integers and does something with it.

!\[\[31\_Backend\_image007.png]]

&#x20;

001 has the longest possible ch count.

!\[\[31\_Backend\_image008.png]]

&#x20;

We get this...of which case we are specified as a superuser.

!\[\[31\_Backend\_image009.png]]

&#x20;

Any other numbers seem to point us to nothing.

I tried again with other parameters with wfuzz.

This time, I wanted to see what other directories with other methods exist.

!\[\[31\_Backend\_image0010.png]]

&#x20;

/login:

!\[\[31\_Backend\_image0011.png]]

&#x20;

There are some stuff that is needed.

Let's try posting some JSON to see what happens:

&#x20;

!\[\[31\_Backend\_image0012.png]]

&#x20;

So there's a body and password needed.

We can eventually sign up using this:

!\[\[31\_Backend\_image0013.png]]

&#x20;

We can then try to login, finding out that JSON does not work and we have to use regular POST data to make it work.

!\[\[31\_Backend\_image0014.png]]

&#x20;

We get this JWT token.

Decoded:

!\[\[31\_Backend\_image0015.png]]

&#x20;

So we get this, and we can see this is an access\_token of some sort.

When we gobusted earlier, we found a /docs directory that cannot be ignored:

&#x20;

/docs:

!\[\[31\_Backend\_image0016.png]]

&#x20;

We can try to use the token we received to authenticate:

!\[\[31\_Backend\_image0017.png]]

&#x20;

It kind of works:

!\[\[31\_Backend\_image0018.png]]

&#x20;

When filtering requests, we can see how there is a second request to access openapi.json

!\[\[31\_Backend\_image0019.png]]

&#x20;

!\[\[31\_Backend\_image0020.png]]

&#x20;

We can see how there is access after using the same header.

Now, we can read more about the type of API this is using and what other endpoints it has.

&#x20;

This is the most interesting one.

!\[\[31\_Backend\_image0021.png]]

&#x20;

This endpoint lets us run a command as the administrator.

Let's try to spoof the administrator cookie. But before we do that, we need the JWT token secret.

&#x20;

!\[\[31\_Backend\_image0022.png]]

&#x20;

We can see how this uses a PUT verb.

!\[\[31\_Backend\_image0023.png]]

&#x20;

This does nothing for us.

&#x20;

Update passwd:

!\[\[31\_Backend\_image0024.png]]

&#x20;

For this, we can see it requires the GUID and newpassword.

!\[\[31\_Backend\_image0025.png]]

&#x20;

We can change the password of the administrator:

!\[\[31\_Backend\_image0026.png]]

&#x20;

Then we can steal his token:

!\[\[31\_Backend\_image0027.png]]

&#x20;

!\[\[31\_Backend\_image0028.png]]

&#x20;

Some key is needed before we can access the debug portion.

&#x20;

Other endpoints:

&#x20;

We can retrieve files with this token:

!\[\[31\_Backend\_image0029.png]]

&#x20;

Let's think, now we have LFI. What can we do from here?

We can look at the environment file.

&#x20;

!\[\[31\_Backend\_image0030.png]]

&#x20;

We can see the app home being within the /home/htb/uhc file.

Combined with the fact that this server is a python server, there is likely some kind of main.py or app.py somewhere.

&#x20;

App.py:

!\[\[31\_Backend\_image0031.png]]

&#x20;

We can convert this to easily read code and then view the stuff.

!\[\[31\_Backend\_image0032.png]]

&#x20;

There's some custom programs here, particularly from app.api.v1.api where it imports api\_router.

&#x20;

We can find yet another endpoint here:

!\[\[31\_Backend\_image0033.png]]

&#x20;

{width="8.40625in" height="1.5208333333333333in"}

&#x20;

Then we can see there's this custom function as well, and it seems to use fastapi APIRouter module to do something...

&#x20;

Let's take a look at the admin.py and stuff.

&#x20;

!\[\[31\_Backend\_image0035.png]]

&#x20;

/exec function:

!\[\[31\_Backend\_image0036.png]]

&#x20;

We can see that the debug thingy is from this function called Depends...?

&#x20;

We should take a look at the deps.py.

!\[\[31\_Backend\_image0037.png]]

&#x20;

{width="4.84375in" height="1.875in"}

&#x20;

There's the JWT\_SECRET!

So for now we need look at the config.py.

!\[\[31\_Backend\_image0039.png]]

&#x20;

!\[\[31\_Backend\_image0040.png]]

Found the secret!

We can now forge the token.

&#x20;

!\[\[31\_Backend\_image0041.png]]

&#x20;

!\[\[31\_Backend\_image0042.png]]

Cool!

&#x20;

Shell:

We can use $IFS or %20 to specify a space.

!\[\[31\_Backend\_image0043.png]]

&#x20;

!\[\[31\_Backend\_image0044.png]]

&#x20;

PE:

Within the user directory, there's this auth.log with a weird username:

!\[\[31\_Backend\_image0045.png]]

&#x20;

That's the root password

!\[\[31\_Backend\_image0046.png]]

&#x20;

Flag:

!\[\[31\_Backend\_image0047.png]]

&#x20;

&#x20;
