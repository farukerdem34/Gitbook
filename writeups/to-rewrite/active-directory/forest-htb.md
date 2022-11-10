# Forest (HTB)

Forest (HTB)

Saturday, 26 February 2022

7:08 pm

This is a Windows AD machine.

The target IP is 10.10.10.161.

My IP is 10.10.16.9.

&#x20;

Take note, I will be going more in-depth for this machine and I'm going to look through walkthroughs so I can develop my methodology when it comes to AD.

!\[\[05\_Forest (HTB)\_image001.png]]

These are generally the ports that will be open for AD-related machines.

&#x20;

Not all the ports are important for our enumeration, hence we will be using these ports to scan instead.

!\[\[05\_Forest (HTB)\_image002.png]]

Now we have identified, firstly, a DNS server.

&#x20;

Let's try by doing some domain lookups of that server to enumerate any other domains that exist on this network.

!\[\[05\_Forest (HTB)\_image003.png]]

&#x20;

!\[\[05\_Forest (HTB)\_image004.png]]

Now, we can see there's one domain that we have found relating to this.

&#x20;

Next thing is to try a zone transfer, of which it does not let me.

Let's now try to enumerate any shares that are found on the network using a Null Session.

This produces a lot of details, and we need to know how to filter this out.

{width="9.854166666666666in" height="6.15625in"}

Looks like these are my users.

&#x20;

{width="5.25in" height="5.802083333333333in"}

These are my groups present on the machine.

{width="5.96875in" height="1.40625in"}

More groups.

&#x20;

That's about it, so we have some users to work with.

There are two accounts to take note of:

&#x20;

* Administrator
* Svc-alfredo

&#x20;

These are the superusers of the machine and what will get us the flags. I created a quick text file with all of the possible hosts that I have found.

!\[\[05\_Forest (HTB)\_image008.png]]

We can use this to perhaps brute force into the machine somehow.

&#x20;

Take note of the domain name that we are trying to break into:

{width="2.7916666666666665in" height="1.0208333333333333in"}

HTB is the main one we want to target.

&#x20;

From this point, we have a few things that we are able to try:

1. Getting hashes for these users
2.  Kerberoasting

    a. Will not work as we do not have any password credentials

&#x20;

These are the two things that come to mind. The rest of the possible methods are on HackTricks.

For now, let's try number 1.

!\[\[05\_Forest (HTB)\_image0010.png]]

How this tool works is by sending AS-REQ to the DC on the behalf of any users, and receive a reply. This last kind of message would contain a data encrypted with the original user key derived from their password. The user password can be cracked offline.

&#x20;

Why this is good is because:

1. We have no accounts
2. We have no credentials
3. This would involve enumerating credentials of which can get a foothold from.

&#x20;

Let's take a look into it.

{width="13.15625in" height="6.03125in"}

This is the tool we will be using.

&#x20;

{width="13.270833333333334in" height="2.125in"}

&#x20;

&#x20;

Interesting, we got a hash out.

Running john, we get this.

{width="11.84375in" height="1.96875in"}

d

Cool!

&#x20;

Now, Windows is a bit effy with SSH, so there's this other tool we can use. It's called Evil-Win-RM.

&#x20;

Back to the port scan, we can see in the very first one I did, there is actually a WinRM port that is open.

&#x20;

!\[\[05\_Forest (HTB)\_image0014.png]]

And just like that, we are dropped into a PowerShell cmd.exe.

&#x20;

From here, we can go ahead and grab that user flag.

&#x20;

Before I start PE, I want to use PowerShell and begin to enumerate out certain things.

&#x20;

!\[\[05\_Forest (HTB)\_image0015.png]]

According to HackTricks, it seems that this tool called BloodHound is one of the best for further AD enumeration once we have gotten a foothold.

&#x20;

!\[\[05\_Forest (HTB)\_image0016.png]]

Seems that we need to make use of this tool called SharpHound, so let's give it a try.

!\[\[05\_Forest (HTB)\_image0017.png]]

From here, it seems that the collectors given to me did not work out well, .exe files do not seem to transfer over easily.

&#x20;

Luckily, there exists a SharpHound.ps1 file which we can use.

&#x20;

!\[\[05\_Forest (HTB)\_image0018.png]]

Afterwards, we just run the tool.

&#x20;

This would go on for a while and sooner or later generate a .zip file.

!\[\[05\_Forest (HTB)\_image0019.png]]

Transfer it back to my account and from there open it on Bloodhound.

!\[\[05\_Forest (HTB)\_image0020.png]]

&#x20;

{width="9.625in" height="5.625in"}

Transfer it over.

Afterwards, we can import this into Bloodhound to view the information in a visual representation.

&#x20;

!\[\[05\_Forest (HTB)\_image0022.png]]

&#x20;

We can see that the path to the domain admin on the right has to go through the WriteDacl path right in the middle.

&#x20;

Our account is a member of a lot of different groups, and we are able to modify and control the group membership of any group.

&#x20;

!\[\[05\_Forest (HTB)\_image0023.png]]

This is what I got from BloodHound, and the attack is clear.

There's even an example down there.

&#x20;

So we can first start with adding a user.

!\[\[05\_Forest (HTB)\_image0024.png]]

&#x20;

Afterwards, we can proceed to add this user to the Exchange Windows Permissions group.

!\[\[05\_Forest (HTB)\_image0025.png]]

&#x20;

Afterwards, we need to add PowerView into the machine in order to start the exploitation, according to the example written below there.

&#x20;

From here, I need to basically add my newly created User into the DCSync place, which is basically adding a user into the Domain Controller.

!\[\[05\_Forest (HTB)\_image0026.png]]

Found these commands online, and used them.

&#x20;

This would basically add me inside,

&#x20;

Afterwhich, we can simply use secretsdump.py to get into the account.

!\[\[05\_Forest (HTB)\_image0027.png]]

This command would dump out the hashes.

&#x20;

We can the just pass the hash to log in.

{width="13.15625in" height="2.0104166666666665in"}

Pwned.

&#x20;

This box was done with the help of a walkthrough because of the fact that I was completely new and did not know what to do when it comes to AD.

&#x20;

Cheers!
