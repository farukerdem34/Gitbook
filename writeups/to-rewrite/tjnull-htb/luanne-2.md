# Luanne

Luanne

Tuesday, 1 March 2022

9:51 pm

This is a Linux machine.

The target IP is 10.10.10.218.

My IP is 10.10.16.9.

&#x20;

Enumeration!

Take note that this is a BSD box, so there'll be some weird stuff here and there.

!\[\[64\_Luanne\_image001.png]]

Firewalled...

Anyways I tried visiting the IP anyways on port 80 using the browser.

I was greeted with this.

{width="5.395833333333333in" height="2.90625in"}

Port 3000...Another reverse proxy.

Did a directory enumeration first while waiting for the Nmap scan.

&#x20;

This comes after asking for credentials as shown:

&#x20;

!\[\[64\_Luanne\_image003.png]]

Odd hint, let's take a look at port 9001.

&#x20;

!\[\[64\_Luanne\_image004.png]]

Another hint, but this time it's asking for default credentials. Now I was sure that I needed that nmap scan to return so I can know what kind of engine this is running before I begin to try default credentials.

&#x20;

At this point, it was taking way too long to enumerate the ports, so I just searched for port 22, 80, 9001, the defaults open on Linux machines. Take note to not do this.

&#x20;

!\[\[64\_Luanne\_image005.png]]

Supervisor process manager...default credentials searching time.

Using the documentation for this engine, I found this:

!\[\[64\_Luanne\_image006.png]]

&#x20;

Let's proceed with this.

I managed to get into this.

!\[\[64\_Luanne\_image007.png]]

&#x20;

!\[\[64\_Luanne\_image008.png]]

Things to take note of.

So there were 3 different links to click on, and I investigated each of them.

!\[\[64\_Luanne\_image009.png]]

This told me well, uptime and that's all.

&#x20;

Processes told me this:

!\[\[64\_Luanne\_image0010.png]]

&#x20;

So I can see that there are some processes, particularly that reverse proxy on port 3000.

There was this weather directory, so I decided to run a full directory enumeration on this.

!\[\[64\_Luanne\_image0011.png]]

&#x20;

Robots.txt.

!\[\[64\_Luanne\_image0012.png]]

Well, nothing much, but I can tell that there is something going in regarding that webapi directory.

The last file revealed this.

!\[\[64\_Luanne\_image0013.png]]

I see false hits and stuff, perhaps web cache poisoning or something else?

&#x20;

Anyways, I investigated that webapi/weather thing.

&#x20;

I did another directory scan on /weather.

Found this.

!\[\[64\_Luanne\_image0014.png]]

&#x20;

Perhaps this is the path towards the web API manipulation step.

!\[\[64\_Luanne\_image0015.png]]

Interesting, this is JSON.

&#x20;

Let's try to append the city=list thing.

&#x20;

!\[\[64\_Luanne\_image0016.png]]

&#x20;

Let's try one of the cities.

!\[\[64\_Luanne\_image0017.png]]

Well this would give me a whole lot of stuff.

&#x20;

&#x20;

Perhaps I could run a wfuzz on this API.

!\[\[64\_Luanne\_image0018.png]]

&#x20;

!\[\[64\_Luanne\_image0019.png]]

Now sure if this matters to me. Turns out it did not matter at all.

&#x20;

Ran a few more LFI tests before some other fuzzes, did not do me well.

&#x20;

!\[\[64\_Luanne\_image0020.png]]

Now I was thinking, could I perhaps escape from this?

!\[\[64\_Luanne\_image0021.png]]

It looks like I'm outside of it.

&#x20;

!\[\[64\_Luanne\_image0022.png]]

I seem to have removed one of the quotes.

!\[\[64\_Luanne\_image0023.png]]

Now, I knew I was out of the thingy. Now I just needed to have some form of proof of concept back to me. Like a ping.

This was written in Lua, which is a programming language. Let's try to call some form of system functions.

&#x20;

!\[\[64\_Luanne\_image0024.png]]

I was getting closer. I found out the comment function. Now I was just getting lua errors...

&#x20;

!\[\[64\_Luanne\_image0025.png]]

Oops.

I changed the command to id, in case ping was not present on the web server (somehow)

!\[\[64\_Luanne\_image0026.png]]

This told me I basically escaped completely from the JSON object.

A bit more later, this occurred after reading documentation properly.

&#x20;

!\[\[64\_Luanne\_image0027.png]]

We now have RCE. To confirm it, I pinged myself.

Did not work, so instead I just reverse shelled.

!\[\[64\_Luanne\_image0028.png]]

&#x20;

!\[\[64\_Luanne\_image0029.png]]

This worked, but I did not gain a shell. A bit more tinkering is required and I should be good to go.

&#x20;

!\[\[64\_Luanne\_image0030.png]]

&#x20;

!\[\[64\_Luanne\_image0031.png]]

Great! We are now \_httpd.

Remember we still have that port 80 to look at.

&#x20;

Interestingly, we are allowed to view the index.html that was located there.

!\[\[64\_Luanne\_image0032.png]]

Even more interestingly, there was this.

!\[\[64\_Luanne\_image0033.png]]

Alright, time to take this and crack it as usual.

&#x20;

{width="9.947916666666666in" height="2.6979166666666665in"}

Cracked.

!\[\[64\_Luanne\_image0035.png]]

Let's try to su.

Does not work, but on the bright side we can now access the port 80 website properly.

!\[\[64\_Luanne\_image0036.png]]

&#x20;

Ok... Now let's take a look properly.

Directory enumeration has no point, we already saw it.

Interestingly, when viewing the stuff, there's this basic authorization in base64.

&#x20;

!\[\[64\_Luanne\_image0037.png]]

This would be just the web page and the password we already found.

But remember, oddly this page was reverse proxied to port 3000.

&#x20;

Anyways, this has no use to me.

We still have this odd command, which I bet is running under the user.

!\[\[64\_Luanne\_image0038.png]]

I needed to find out the processes that were running on this machine.

!\[\[64\_Luanne\_image0039.png]]

He was running the same command, this time on port 3001!

&#x20;

Curl exists on this machine, perhaps we could do the same thing?

{width="16.15625in" height="1.1666666666666667in"}

This didn't work as planned...

&#x20;

!\[\[64\_Luanne\_image0041.png]]

Interestingly, when I tried going to the home directory from this website, it gave me an item is not found instead...

!\[\[64\_Luanne\_image0042.png]]

This means I can probably access something within this.

&#x20;

Now, looking at curl manual. I can specify username and password with the -u flag.

&#x20;

Went for a shot in the dark.

!\[\[64\_Luanne\_image0043.png]]

As you can see, this gave me something cool.

&#x20;

!\[\[64\_Luanne\_image0044.png]]

User flag done!

Grab that user flag.

&#x20;

Within my home directory, there's this backups directory with this.

!\[\[64\_Luanne\_image0045.png]]

This is encrypted, and I have little to no idea how to decrypt it.

&#x20;

Interestingly, there's this file within the home directory that also has this pub and secring files.

!\[\[64\_Luanne\_image0046.png]]

These are PGP key things, and I have to find a software of which to decrypt them probably.

Gpg was not on the machine.

!\[\[64\_Luanne\_image0047.png]]

Tried this openpgp tool, as I thought maybe it'll be something.

Didn't work. So I went to the bin folder to see whatever was in this damn machine.

!\[\[64\_Luanne\_image0048.png]]

Netpgp?

&#x20;

{width="6.28125in" height="5.53125in"}

There we go.

&#x20;

Now we need to decrypt this thing somehow.

&#x20;

!\[\[64\_Luanne\_image0050.png]]

This should work right?

&#x20;

!\[\[64\_Luanne\_image0051.png]]

&#x20;

At first it looked the same, except the .htpasswd was a different kind of hash.

&#x20;

{width="9.979166666666666in" height="2.84375in"}

Pop it open.

&#x20;

This should be the password of root or something else, but freebsd does not have sudo.

&#x20;

!\[\[64\_Luanne\_image0053.png]]

Doas?

!\[\[64\_Luanne\_image0054.png]]

Welp.

&#x20;

That was pretty fun.

&#x20;

&#x20;

&#x20;
