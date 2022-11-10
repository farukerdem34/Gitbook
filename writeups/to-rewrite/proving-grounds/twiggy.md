# Twiggy

Twiggy

Wednesday, 16 March 2022

9:06 pm

The machine IP is 192.168.159.62.

My IP is 192.168.49.159.

&#x20;

!\[\[11\_Twiggy\_image001.png]]

Not too sure what ZMTP is, and port 8000 is just one JSON format there, not sure what that means.

&#x20;

!\[\[11\_Twiggy\_image002.png]]

&#x20;

The main port 80 contains this website:

!\[\[11\_Twiggy\_image003.png]]

&#x20;

Within the website, we can see that there is some Blog, and when clicked, it has two links:

!\[\[11\_Twiggy\_image004.png]]

&#x20;

Atom is nothing, but RSS contains this:

!\[\[11\_Twiggy\_image005.png]]

Okay, back to port 8000.

&#x20;

I thought about this JSON for a long time:

!\[\[11\_Twiggy\_image006.png]]

&#x20;

I looked again, and found this:

!\[\[11\_Twiggy\_image007.png]]

&#x20;

!\[\[11\_Twiggy\_image008.png]]

Salt API?

&#x20;

Looks like I found the right one.

I got my exploit from [here](https://github.com/jasperla/CVE-2020-11651-poc) and installed salt api onto my machine.

&#x20;

Following the exploit, I managed to get the root key.

!\[\[11\_Twiggy\_image009.png]]

&#x20;

To avoid firewalls, send the reverse shell to port 80.

&#x20;

!\[\[11\_Twiggy\_image0010.png]]

&#x20;

!\[\[11\_Twiggy\_image0011.png]]

&#x20;
