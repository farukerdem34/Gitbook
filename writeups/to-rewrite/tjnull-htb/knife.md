# Knife

Knife

Thursday, 24 February 2022

9:09 pm

This is a Linux machine.

My IP is 10.10.16.9.

The target IP is 10.10.10.242.

&#x20;

As always.

!\[\[56\_Knife\_image001.png]]

&#x20;

I also ran a directory scan on the entire website and a vuln script from here.

Nothing. Went around for a while before deciding to take a look at Burpsuite traffic.

Noticed this!

!\[\[56\_Knife\_image002.png]]

PHP/8.1.0-dev?

&#x20;

There's one exploit for this.

!\[\[56\_Knife\_image003.png]]

So let's just try it.

&#x20;

That was easy.

!\[\[56\_Knife\_image004.png]]

We are instantly the user.

&#x20;

Do a cat /home/james/user.txt.

&#x20;

It would seem that this particular web shell is a bit...limited.

&#x20;

We would need to break out of this somehow.

&#x20;

Get a reverse shell from this port back to our machine.

!\[\[56\_Knife\_image005.png]]

&#x20;

!\[\[56\_Knife\_image006.png]]

&#x20;

!\[\[56\_Knife\_image007.png]]

Stabilise using python3 and we are on our way.

Let's check sudo -l.

&#x20;

Turns out we can run knife.

&#x20;

!\[\[56\_Knife\_image008.png]]

Based on GTFOBins, do this command.

!\[\[56\_Knife\_image009.png]]

Pwned!
