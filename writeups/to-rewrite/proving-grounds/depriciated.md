# Depriciated

Depriciated

Saturday, 19 March 2022

9:11 pm

Interestingly, this is a graphQL Data Query language that is being used here.

&#x20;

Very interesting, take a read at Hack tricks and stuff.

Go to payloadallthethings and get some commands that can be run on this.

&#x20;

Gave up because I had no idea what I was doing, and looked at a hint. It points towards this:

!\[\[23\_Depriciated\_image001.png]]

Alright, we have some users.

&#x20;

From here, we can take a look at the PayloadAllTheThings payload to enumerate this:

!\[\[23\_Depriciated\_image002.png]]

This would take one user.

&#x20;

From here, we can directly look at the OTP:

!\[\[23\_Depriciated\_image003.png]]

&#x20;

With this, we can access the port on 5132, which is a CLI.

!\[\[23\_Depriciated\_image004.png]]

From here, we can read some tickets and stuff:

!\[\[23\_Depriciated\_image005.png]]

&#x20;

So peter has a weakpassword.

Turns out the password is peter@safe.

&#x20;

!\[\[23\_Depriciated\_image006.png]]

&#x20;

From here, we can grab the user flag.

&#x20;

Really interesting, never played with this API before and was stumped, until I realised that what we did beforehand was correct, just that I didn't know what I was reading.

&#x20;

Ran a winpeas from here and began my enumeration.

Didn't help, but there was a file within /opt/depreciated/messaging.

&#x20;

Inside the message.py file, there was this interesting bit:

{width="10.020833333333334in" height="1.125in"}

User=admin?

From here, we can see that it takes usernames from code.txt:

!\[\[23\_Depriciated\_image008.png]]

The answer is clear, we just need to echo in a username as admin with some password and be able to view more messages.

&#x20;

Messages are taken from msg.json, which is interestingly not open for us to read:

!\[\[23\_Depriciated\_image009.png]]

&#x20;

&#x20;

Interestingly, there is a create\_message thing, whereby we can directly change files and stuff too:

!\[\[23\_Depriciated\_image0010.png]]

&#x20;

So in order to exploit this, we would just need to create a malicious code.txt file with another admin within it that has a password of our choosing, and then proceed to get our m

!\[\[23\_Depriciated\_image0011.png]]

alicious text file as the main code.txt file there.

!\[\[23\_Depriciated\_image0012.png]]

Great.

&#x20;

Within this, there would be a new ticket with number 0 for it:

!\[\[23\_Depriciated\_image0013.png]]

&#x20;

With this password, we can SSH in as root.

&#x20;

Not very realistic, but more puzzle and challenges.

&#x20;

!\[\[23\_Depriciated\_image0014.png]]

&#x20;
