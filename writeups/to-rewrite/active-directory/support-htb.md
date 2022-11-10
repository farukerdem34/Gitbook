# Support (HTB)

Support (HTB)

Thursday, August 18, 2022

9:44 AM

Nmap scan:

!\[\[22\_Support (HTB)\_image001.png]]

&#x20;

SMBMap Anonymous Session:

!\[\[22\_Support (HTB)\_image002.png]]

&#x20;

Share:

!\[\[22\_Support (HTB)\_image003.png]]

&#x20;

Interesting information here. There is this one UserInfo.exe.zip that I'm interested in.

Within this, there is something else.

!\[\[22\_Support (HTB)\_image004.png]]

&#x20;

We can strings the binary to find out more about it.

!\[\[22\_Support (HTB)\_image005.png]]

&#x20;

So the user is called 0xdf, which is the creator of this particular machine.

Perhaps we need to reverse engineer this thing.

&#x20;

When running the binary, we can find that it is doing LDAP queries.

!\[\[22\_Support (HTB)\_image006.png]]

&#x20;

Which is interesting, because it looks like it was meant to run locally or something.

We can port this over to a Windows VM to begin some analysis.

&#x20;

We can then use ILSpy to decompile and begin to read source code through VS Code.

&#x20;

!\[\[22\_Support (HTB)\_image007.png]]

&#x20;

We see stuff like this, and there is some form of password going around.

&#x20;

We can also find some form of key here.

!\[\[22\_Support (HTB)\_image008.png]]

&#x20;

So basically, we can just follow this logic and then decode the password out.

We then know the user is called support as well, based on the earlier thing found.

&#x20;

Through some magic, we can decode this easily.

We just need to replicate this function within python or something.

&#x20;

!\[\[22\_Support (HTB)\_image009.png]]

&#x20;

!\[\[22\_Support (HTB)\_image0010.png]]

&#x20;

We can then use this password to begin enumerating LDAP again.

&#x20;

LDAP Enumeration:

!\[\[22\_Support (HTB)\_image0011.png]]

&#x20;

Within this, we can find a password.

!\[\[22\_Support (HTB)\_image0012.png]]

&#x20;

With this, we can login as support.

!\[\[22\_Support (HTB)\_image0013.png]]

&#x20;

Flag:

!\[\[22\_Support (HTB)\_image0014.png]]

&#x20;

PE:

We can do some basic enumeration using bloodhound and the likes.

&#x20;

We can see that the support user is part of this group here, which may give it permissions.

!\[\[22\_Support (HTB)\_image0015.png]]

&#x20;

This is something that definitely needs Bloodhound to map out.

&#x20;

!\[\[22\_Support (HTB)\_image0016.png]]

&#x20;

And from bloodhound, we can find this.

!\[\[22\_Support (HTB)\_image0017.png]]

&#x20;

Cool!

We would need some powermad and powerview to abuse this.

&#x20;

First, we need to create a new Machine account with some fake name and password.

&#x20;

!\[\[22\_Support (HTB)\_image0018.png]]

&#x20;

Then, we can retrieve the SID of this account.

&#x20;

Afterwards, we need to create a new raw security descriptor.

!\[\[22\_Support (HTB)\_image0019.png]]

&#x20;

Then we would need to set the security descriptor bytes to the object.

&#x20;

!\[\[22\_Support (HTB)\_image0020.png]]

&#x20;

We can check the privileges here.

!\[\[22\_Support (HTB)\_image0021.png]]

&#x20;

Then, we can use Rubeus.exe to dump out the hash of the administrator.

&#x20;

First, generate the required hash for the account we control.

!\[\[22\_Support (HTB)\_image0022.png]]

&#x20;

!\[\[22\_Support (HTB)\_image0023.png]]

&#x20;

Then, we can impersonate the administrator!

!\[\[22\_Support (HTB)\_image0024.png]]

&#x20;

!\[\[22\_Support (HTB)\_image0025.png]]

&#x20;

However, this generates some form of error when trying to inject this ticket into the klist memory.

As it turns out, we need to enable impersonation on the administrator user.

&#x20;

!\[\[22\_Support (HTB)\_image0026.png]]

&#x20;

This would allow us to request a ticket as the administrator.

&#x20;

!\[\[22\_Support (HTB)\_image0027.png]]

&#x20;

We can then use psexec or something to gain access to the machine.

!\[\[22\_Support (HTB)\_image0028.png]]

&#x20;

!\[\[22\_Support (HTB)\_image0029.png]]

&#x20;

Flag:

!\[\[22\_Support (HTB)\_image0030.png]]

&#x20;

&#x20;
