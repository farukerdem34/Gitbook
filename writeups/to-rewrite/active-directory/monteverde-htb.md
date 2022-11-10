# Monteverde (HTB)

Monteverde (HTB)

Saturday, 12 March 2022

1:54 pm

This is a Windows AD machine.

The target IP is 10.10.10.172

My IP is 10.10.16.9.

&#x20;

Let's enumerate this thing to see what ports and services are running.

Decided to run both an Nmap and a Enum4linux just to see what I could find.

Saves time too.

&#x20;

Nmap:

!\[\[08\_Monteverde (HTB)\_image001.png]]

Most interestingly, there was a DNS port there. Let's investigate further with a version scan.

!\[\[08\_Monteverde (HTB)\_image002.png]]

&#x20;

Enum4linux allowed for Null sessions!

{width="10.979166666666666in" height="2.625in"}

There are some users here.

Some groups as well:

{width="10.885416666666666in" height="6.25in"}

&#x20;

Alright, we have some users now.

There's also one domain we are interested in:

{width="2.5104166666666665in" height="0.96875in"}

&#x20;

Let's dump all the users into one file and then proceed with some hash dumping.

!\[\[08\_Monteverde (HTB)\_image006.png]]

Hash dumping did not work, so time to brute force SMB.

!\[\[08\_Monteverde (HTB)\_image007.png]]

&#x20;

And we found one!

&#x20;

!\[\[08\_Monteverde (HTB)\_image008.png]]

&#x20;

{width="10.65625in" height="3.0in"}

Alright, there's an E drive, which indicates something that we might have to look into later. Let's look at the users$ file because that's the most interesting to me.

&#x20;

{width="9.25in" height="3.9479166666666665in"}

Right, so let's just get that azure.xml file because it's the only thing that's present on this. Also indicates that we are on an Azure AD machine.

&#x20;

!\[\[08\_Monteverde (HTB)\_image0011.png]]

This comes with more credentials, presumably for the mhope user.

&#x20;

!\[\[08\_Monteverde (HTB)\_image0012.png]]

Ran a psexec and it did not work, so I ran another evil-winrm just to check whether the user was part of the Remote User group.

!\[\[08\_Monteverde (HTB)\_image0013.png]]

Great!

Ran a Winpeas to check on stuff, and the E drive contained nothing of interest to me, just the other azure.xml which I found earlier.

&#x20;

{width="6.364583333333333in" height="1.875in"}

&#x20;

!\[\[08\_Monteverde (HTB)\_image0015.png]]

Didn't give much, so I took a look at the Memberships this user was part of:

!\[\[08\_Monteverde (HTB)\_image0016.png]]

Azure admin...So I'm kind of a privileged user on the AD network. This would mean that I'm able to upload files right? Within the E drive, there was this:

!\[\[08\_Monteverde (HTB)\_image0017.png]]

There was this uploads file, but I'm not too sure what I can do from that.

At this point, I began looking at DCSync attacks, because I was a privileged user.

&#x20;

So from here, we can view the winpeas output again:

!\[\[08\_Monteverde (HTB)\_image0018.png]]

We can modify these programs that have been installed.

&#x20;

This abuses the Azure AD connect, which is allows us to sync our password or pass the cloud authenticate to our internal AD. So we can sync our passwords across multiple services that are being run on the machine.

&#x20;

Didn't know what I was doing and looked up a walkthrough, which brought me to this [blog](https://blog.xpnsec.com/azuread-connect-for-redteam/):

There's a POC script there and when done, reveals this:

&#x20;

!\[\[08\_Monteverde (HTB)\_image0019.png]]

&#x20;

!\[\[08\_Monteverde (HTB)\_image0020.png]]

From here, we can pwn the machine.

&#x20;

But first, why does this work?

&#x20;

Basically, our user mhope is able to handle replication of AD to Azure, as he is the Azure admin. In the default case, there would be a user like MSOL\_hex, but it's not here.

Anyways, we are able to connect to the local database and pull the configuration that is present there.

So our script is able to connect to the database and get it to decrypt the username and password for the account that handles the replication.

&#x20;

I would not have been able to figure this out alone...

Great box though.

&#x20;

&#x20;
