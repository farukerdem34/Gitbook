# Sorcerer

Sorcerer

Wednesday, 16 March 2022

4:46 pm

The target IP is 192.168.61.100.

My IP is 192.168.49.61.

&#x20;

!\[\[07\_Sorcerer\_image001.png]]

&#x20;

!\[\[07\_Sorcerer\_image002.png]]

&#x20;

War file time.

&#x20;

Now, the goal is to determine some possible credentials to log in or something.

&#x20;

Port 80 had nothing, just a 404.

Port 7742 gave me this website:

!\[\[07\_Sorcerer\_image003.png]]

&#x20;

And port 8080 gave me this website:

!\[\[07\_Sorcerer\_image004.png]]

There's no sign in prompt, and there are the usual default tomcat credentials shown when we click on Manager App or something.

&#x20;

!\[\[07\_Sorcerer\_image005.png]]

&#x20;

These credentials do not work however.

Anyways, port 111 showed some potential when it retuened some stuff.

&#x20;

!\[\[07\_Sorcerer\_image006.png]]

These were the services that were running on it.

&#x20;

Port 2049 had nothing to mount, however.

&#x20;

Right, so we should be using some form of directory enumeration, since all other avenues have been exhausted.

&#x20;

After a long time, this was picked up by the gobuster search.

!\[\[07\_Sorcerer\_image007.png]]

&#x20;

On this, there were 4 different zip files.

!\[\[07\_Sorcerer\_image008.png]]

&#x20;

Unzipping them, it seems there are linux home directories on it.

!\[\[07\_Sorcerer\_image009.png]]

&#x20;

Only max was interesting.

!\[\[07\_Sorcerer\_image0010.png]]

&#x20;

From here, we can actually get the tomcat users and also his private key.

Also, it seems that max blocks SSH-ing into his machine using his key.

&#x20;

{width="8.854166666666666in" height="1.8645833333333333in"}

Why is this so?

&#x20;

Well, because user max has a thing on his directory that blocks SSH but allows for SCP.

!\[\[07\_Sorcerer\_image0012.png]]

&#x20;

So what we need to do is somehow, transfer this same file over to max and then SSH in.

We need to alter it first.

!\[\[07\_Sorcerer\_image0013.png]]

Cat scp

{width="11.072916666666666in" height="0.84375in"}

&#x20;

!\[\[07\_Sorcerer\_image0015.png]]

Then we can SSH in!

The local.txt is within the user dennis.

!\[\[07\_Sorcerer\_image0016.png]]

&#x20;

&#x20;

Ran a linpeas.sh and saw this:

!\[\[07\_Sorcerer\_image0017.png]]

GTFObins this, and we can break out of shells apparently.

&#x20;

!\[\[07\_Sorcerer\_image0018.png]]

&#x20;

!\[\[07\_Sorcerer\_image0019.png]]

&#x20;

!\[\[07\_Sorcerer\_image0020.png]]

Pwned.

&#x20;
