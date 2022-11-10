# Pandora

Pandora

Monday, 28 March 2022

4:56 pm

A Linux machine with these ports.

!\[\[08\_Pandora\_image001.png]]

&#x20;

!\[\[08\_Pandora\_image002.png]]

Great.

&#x20;

Viewing the web port, we see a nice little website.

!\[\[08\_Pandora\_image003.png]]

Add that to our hosts, and let's run a gobuster on this to enumerate further.

&#x20;

At the contact portion, there are 2 emails to take note of.

!\[\[08\_Pandora\_image004.png]]

We may or may not need these.

&#x20;

Anyways the website had nothing, so I did a UDP scan to check on what is present on the machine.

Pretty much had a hunch for SNMPwalk being useful here, because UDP ports are always something that no one checks on.

!\[\[08\_Pandora\_image005.png]]

There were these things I saw at the start, and that looks like a password of some sort.

&#x20;

Also, I found out this was running smnpv2.

!\[\[08\_Pandora\_image006.png]]

&#x20;

!\[\[08\_Pandora\_image007.png]]

Within the stuff, I found this.

&#x20;

With that, I can SSH in as daniel.

&#x20;

!\[\[08\_Pandora\_image008.png]]

The user flag is with the other user called Matt.

&#x20;

At the very least, I could see the other websites and hosts that were available on this server.

&#x20;

Ran a linpeas for this, and determined that Nmap was on the machine. Perhaps we need to do some pivoting.

!\[\[08\_Pandora\_image009.png]]

Went to this directory.

!\[\[08\_Pandora\_image0010.png]]

&#x20;

However, there was nothing in here for me. I needed to find other websites or stuff that is being hosted on this machine. Or at least find out more credentials.

&#x20;

!\[\[08\_Pandora\_image0011.png]]

Looks like there were more hosts that we could have went to.

&#x20;

!\[\[08\_Pandora\_image0012.png]]

Within this, there was a pandora\_console thing we could get. I tried to get it.

&#x20;

From the looks of that file, it seems that this was a SQL based file that we had to do SQL Injections on.

!\[\[08\_Pandora\_image0013.png]]

&#x20;

We can access it through that URL.

Afterwards, we need to access this through port tunneling I think.

!\[\[08\_Pandora\_image0014.png]]

&#x20;

!\[\[08\_Pandora\_image0015.png]]

This seems to be enabled on the machine, but going to that URL would lead me back to the old one. Time to SSH tunnel.

!\[\[08\_Pandora\_image0016.png]]

We would just tunnel our port 4444 to his port 80.

&#x20;

!\[\[08\_Pandora\_image0017.png]]

This was running Pandora 1.0.0.

!\[\[08\_Pandora\_image0018.png]]

&#x20;

Searchsploited it.

!\[\[08\_Pandora\_image0019.png]]

&#x20;

There were quite a few exploits online too.

&#x20;

!\[\[08\_Pandora\_image0020.png]]

&#x20;

Using this exploit, I was able to login!

!\[\[08\_Pandora\_image0021.png]]

From here, I changed the administrator password to something easy and began looking for authenticated RCEs.

&#x20;

Found this exploit too, which would allow us to bypass the authentication as well.

!\[\[08\_Pandora\_image0022.png]]

This would effectively give us RCE.

&#x20;

Cool!

From here, we can grab the user flag and also try to echo in our SSH key or gain a reverse shell. The former did not work, hence I proceed with the latter.

&#x20;

Used a python3 reverse shell for this one.

!\[\[08\_Pandora\_image0023.png]]

&#x20;

!\[\[08\_Pandora\_image0024.png]]

From here I ran another linpeas to try and see what can this user do.

&#x20;

!\[\[08\_Pandora\_image0025.png]]

We seem to have access to at.

!\[\[08\_Pandora\_image0026.png]]

I found these files aftger like 2 hours. The top one could be executed my me and was owned by root.

&#x20;

Execution and then running another linpeas reveals that there is an unquoted and vulnerable path here.

!\[\[08\_Pandora\_image0027.png]]

&#x20;

We can simply make root execute the tar, which can be a reverse shell or something.

&#x20;

!\[\[08\_Pandora\_image0028.png]]

&#x20;

!\[\[08\_Pandora\_image0029.png]]

&#x20;

!\[\[08\_Pandora\_image0030.png]]

&#x20;

Just like that it's been pwned. That took a long time.

&#x20;

&#x20;
