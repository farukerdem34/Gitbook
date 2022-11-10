# Sauna (HTB)

Sauna (HTB)

Saturday, 12 March 2022

10:03 pm

This is a Windows AD machine.

The target IP is 10.10.10.175

My IP is 10.10.16.9.

&#x20;

Let's enumerate this.

!\[\[09\_Sauna (HTB)\_image001.png]]

Usual ports that we wopuld get from this.

!\[\[09\_Sauna (HTB)\_image002.png]]

Right now we have that domain name which is always useful to find.

Enum4linux reveals nothing.

&#x20;

Kerberos reveals nothing. LDAP enumeration reveals something however.

!\[\[09\_Sauna (HTB)\_image003.png]]

There's one Hugo Smith there, which is definitely not something that is normal. This must be a user on the server somehow.

&#x20;

Website:

!\[\[09\_Sauna (HTB)\_image004.png]]

&#x20;

Anyways this revealed nothing.

&#x20;

I was more curious about the LDAP services, which did return me something.

In the end, ldapsearch was run but nothing was useful.

{width="9.104166666666666in" height="1.6875in"}

&#x20;

Let's view some kerbrute. Running out of ideas here!

{width="11.041666666666666in" height="2.6666666666666665in"}

&#x20;

Now, we wait till it finds something. Looking back at the website, I saw this:

!\[\[09\_Sauna (HTB)\_image007.png]]

One security manager...

&#x20;

Kerbrute gave me these users.

!\[\[09\_Sauna (HTB)\_image008.png]]

&#x20;

While it was running, thought I'd do some AS-REP stuff.

&#x20;

Ran this and it outputted a hash.

{width="9.25in" height="3.3229166666666665in"}

&#x20;

Ran a John on them, and just waited.

!\[\[09\_Sauna (HTB)\_image0010.png]]

Cool!

We can evil-winrm in.

&#x20;

!\[\[09\_Sauna (HTB)\_image0011.png]]

&#x20;

Grab the user flag from this.

!\[\[09\_Sauna (HTB)\_image0012.png]]

Also ran a smbmap on another tab.

&#x20;

{width="10.46875in" height="2.7291666666666665in"}

Well, one share is interesting, the rest not so much.

&#x20;

Got winpeas onto this using our good SMB stuff.

!\[\[09\_Sauna (HTB)\_image0014.png]]

Ran it.

Saw this.

!\[\[09\_Sauna (HTB)\_image0015.png]]

Alright, some more stuff!

Checking the users, there was this one user:

!\[\[09\_Sauna (HTB)\_image0016.png]]

Looks to be close enough.

&#x20;

!\[\[09\_Sauna (HTB)\_image0017.png]]

Right.

&#x20;

Nothing special here, so let's run a bloodhound.

&#x20;

Got SharpHound.exe into the machine.

&#x20;

!\[\[09\_Sauna (HTB)\_image0018.png]]

Ran it, and waited first.

Couldn't get it to work.

&#x20;

I just ran a secretsdump to check whether DCSync attacks work, and it does?? In hindsight, should have use some python bloodhound in order to check on whether this works properly or not.

{width="11.072916666666666in" height="1.9375in"}

Welp.

&#x20;

Now we can just pass the hash.

Just take the second half of that hash and we're in.

!\[\[09\_Sauna (HTB)\_image0020.png]]

&#x20;

&#x20;

&#x20;

&#x20;

&#x20;

&#x20;
