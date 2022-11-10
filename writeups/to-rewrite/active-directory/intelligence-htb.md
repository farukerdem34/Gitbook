# Intelligence (HTB)

Intelligence (HTB)

Friday, 11 March 2022

12:47 pm

This is a Windows AD machine.

The target IP is 10.10.10.248

My IP is 10.10.16.9.

&#x20;

AD enumeration time.

Always start with an Nmap and then proceed with the AD tools.

!\[\[06\_Intelligence (HTB)\_image001.png]]

&#x20;

The usual ports for AD.

Did a detailed version scan on these ports.

&#x20;

!\[\[06\_Intelligence (HTB)\_image002.png]]

Good to know the LDAP stuff, so let's enumerate that Ldap port further.

&#x20;

!\[\[06\_Intelligence (HTB)\_image003.png]]

&#x20;

!\[\[06\_Intelligence (HTB)\_image004.png]]

Add this to our hosts file.

&#x20;

I noticed there was another LDAP port that was up, but running another scan was a bit pointless.

From here, I enumerated the Kerberos port.

!\[\[06\_Intelligence (HTB)\_image005.png]]

Stopping for now as I am too unsure of what to do.

&#x20;

Right, let's continue on.

&#x20;

Let's try to see the website, of which it does have one.

!\[\[06\_Intelligence (HTB)\_image006.png]]

&#x20;

Scrolling down, I was able to find some documents that we could download.

&#x20;

!\[\[06\_Intelligence (HTB)\_image007.png]]

I noticed that there was this date and the format was the same for all of the pdfs. Hence, I googled how to create a wordlist with just this date and came up with this:

&#x20;

!\[\[06\_Intelligence (HTB)\_image008.png]]

From here, we can just use this wordlist to download every possible file from this.

&#x20;

!\[\[06\_Intelligence (HTB)\_image009.png]]

Afterwards, I noticed that each pdf file had its own unique author and stuff.

!\[\[06\_Intelligence (HTB)\_image0010.png]]

From here, we just need to run a quick loop and we should have ourselves a wordlist to use.

&#x20;

!\[\[06\_Intelligence (HTB)\_image0011.png]]

For every single pdf I tried to enumerate something that was english, as most of them were lorep ipsum nonsense.

&#x20;

!\[\[06\_Intelligence (HTB)\_image0012.png]]

This file has one password.

&#x20;

!\[\[06\_Intelligence (HTB)\_image0013.png]]

I checked for user and other english words as well.

&#x20;

!\[\[06\_Intelligence (HTB)\_image0014.png]]

This other file had an english word.

&#x20;

&#x20;

Using this password, we can actually do some password spraying.

&#x20;

!\[\[06\_Intelligence (HTB)\_image0015.png]]

Right, we have this now.

{width="10.635416666666666in" height="2.8541666666666665in"}

&#x20;

The IT disk is interesting.

!\[\[06\_Intelligence (HTB)\_image0017.png]]

Right, so let's just get every file there is.

&#x20;

There's only one though..

!\[\[06\_Intelligence (HTB)\_image0018.png]]

&#x20;

Within this, we are able to see that the machine uses some kind of web cache thingy to resolve host names:

!\[\[06\_Intelligence (HTB)\_image0019.png]]

&#x20;

&#x20;

For now, let's focus on Ted.

!\[\[06\_Intelligence (HTB)\_image0020.png]]

&#x20;

Ted is the user here. After taking the user stuff from each of the files, I was left with a wordlist.

&#x20;

{width="4.3125in" height="1.96875in"}

There was one user called Ted.Graves.

&#x20;

This must be the valid user I need.

First we needed to brute force his smb stuff perhaps.

Put on a rockyou.txt and just left it to run.

&#x20;

For now, we just wait patiently.

Because of how insanely long this was, I knew there was a better way.24

&#x20;

Earlier on, we saw the password policy for this account, which was 7 characters. Additionally, the walkthrough I looked at had used the name 'Ted' within the password to reduce rockyou.txt

&#x20;

!\[\[06\_Intelligence (HTB)\_image0022.png]]

This would take all of the stuff that rockyou.txt has, and only get the ones with 7 characters and the username Ted within it.

&#x20;

Eventually...This worked. Take note that this machine is pretty unrealistic but hey, works.

&#x20;

!\[\[06\_Intelligence (HTB)\_image0023.png]]

&#x20;

Did ab LDAP search, and revealed this.

&#x20;

Looking at the service account, it has certain permissions:

&#x20;

!\[\[06\_Intelligence (HTB)\_image0024.png]]

&#x20;

!\[\[06\_Intelligence (HTB)\_image0025.png]]

Let's try this out.

&#x20;

Damn, we got a hash.

!\[\[06\_Intelligence (HTB)\_image0026.png]]

&#x20;

We need to find out more about this svc\_int user.

WE know the UAC number, which is 16781312

&#x20;

But from there, what kind of permissions do we have?

Googling the numbers, we find this:

&#x20;

!\[\[06\_Intelligence (HTB)\_image0027.png]]

&#x20;

From this TRUSTED\_TO\_AUTH\_FOR\_DELEGATION authority, we can actually steal tickets and impersonate the administrator. This would be called Constrained Delegation.

&#x20;

Followed this [guide](https://hackso.me/intelligence-htb-walkthrough/) for more information about it, and when run, it gives me this:

&#x20;

{width="13.5625in" height="1.3645833333333333in"}

&#x20;

Luckily, we can set our clock to that of the machine.

&#x20;

Run it again after using ntpdate to that of the host.

&#x20;

{width="9.291666666666666in" height="1.0in"}

&#x20;

!\[\[06\_Intelligence (HTB)\_image0030.png]]

Right, now this works.

&#x20;

We now have this ticket.

From here we follow the rest of the exploit.

{width="9.25in" height="3.3854166666666665in"}

And we're in, as the administrator.

&#x20;

!\[\[06\_Intelligence (HTB)\_image0032.png]]

Can grab both flags at once.

&#x20;

&#x20;
