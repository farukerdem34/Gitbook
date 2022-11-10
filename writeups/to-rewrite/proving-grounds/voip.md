# VoIP

VoIP

Saturday, July 9, 2022

12:52 PM

Nmap scan:

!\[\[76\_VoIP \_image001.png]]

&#x20;

Port 80:

Has one login page but we have no credentials.

!\[\[76\_VoIP \_image002.png]]

&#x20;

&#x20;

Port 8000:

It had a login page but we could sign in with admin:admin.

!\[\[76\_VoIP \_image003.png]]

&#x20;

With this page, we can see some administrative details regarding the usage of some VoIP phone dialing:

!\[\[76\_VoIP \_image004.png]]

&#x20;

!\[\[76\_VoIP \_image005.png]]

&#x20;

At this point, we might need to put on earphones to perhaps listen to a password or something.

&#x20;

We can see some logs regarding calls that were happening.

!\[\[76\_VoIP \_image006.png]]

&#x20;

&#x20;

As well as the config of the phone.

{width="9.510416666666666in" height="3.1354166666666665in"}

&#x20;

To find out more, we would need to call this phone, and we know that port 5060 should be open on this.

At the same time, it's a good idea to have wireshark open to capture the SIP packets in between me and the server.

&#x20;

So to do this, we would need to call this phone.

We can use this script to gain a password.

[https://github.com/Pepelux/sippts/wiki/SIPDigestLeak](https://github.com/Pepelux/sippts/wiki/SIPDigestLeak)

!\[\[76\_VoIP \_image008.png]]

&#x20;

And we gain a password, which cracks to 'passion'.

&#x20;

With this, we can login to port 80.

!\[\[76\_VoIP \_image009.png]]

&#x20;

With this, we can find this audio file here.

!\[\[76\_VoIP \_image0010.png]]

&#x20;

Literally only one has any form of volume in it, so let's try to play this.

First of all, we need to decode the .raw file to something else so that it can be played.

&#x20;

A quick check on this reveals the type of file header present.

{width="3.7291666666666665in" height="0.8125in"}

&#x20;

So we have to use some form of rtpplay.

Using rtptools from github, and some StackOverflow, we can do some analysis.

&#x20;

Firstly, we need some of the information regarding the file.

&#x20;

We can find this on the Stream Rates tab within the streams.php file located on the web server.

!\[\[76\_VoIP \_image0012.png]]

&#x20;

So basically, we can take note that this uses 8000Hz frequency. We would need to stream the raw file using some sox.

This would mean that it uses the G.711 encoder pcm\_mulaw to encode the file.

&#x20;

Decoding file:

!\[\[76\_VoIP \_image0013.png]]

&#x20;

Then, we can play this file and hear this one line.

**Your password has been changed to Password1234 where the P is capital.**

&#x20;

Cool, we have a password.

I guessed the username to be something like voiper, and it worked. This allowed us to SSH into the machine.

&#x20;

From there, we can grab the local.txt.

&#x20;

Flag:

!\[\[76\_VoIP \_image0014.png]]

&#x20;

Then we can check sudo permissions for PE.

{width="9.75in" height="1.8645833333333333in"}

&#x20;

And easily grab the root flag.

&#x20;

Flag:

!\[\[76\_VoIP \_image0016.png]]

&#x20;

&#x20;
