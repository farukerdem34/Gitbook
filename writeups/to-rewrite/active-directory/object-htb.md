# Object (HTB)

Object (HTB)

Friday, June 10, 2022

3:56 PM

Nmap scan:

!\[\[19\_Object (HTB)\_image001.png]]

&#x20;

!\[\[19\_Object (HTB)\_image002.png]]

&#x20;

Adding to hosts file:

!\[\[19\_Object (HTB)\_image003.png]]

&#x20;

Port 80 and Port 8080:

!\[\[19\_Object (HTB)\_image004.png]]

&#x20;

Automation server:

!\[\[19\_Object (HTB)\_image005.png]]

&#x20;

Creation of user:

!\[\[19\_Object (HTB)\_image006.png]]

&#x20;

!\[\[19\_Object (HTB)\_image007.png]]

&#x20;

!\[\[19\_Object (HTB)\_image008.png]]

&#x20;

Creating new object:

!\[\[19\_Object (HTB)\_image009.png]]

&#x20;

Freestyle project:

!\[\[19\_Object (HTB)\_image0010.png]]

&#x20;

!\[\[19\_Object (HTB)\_image0011.png]]

Set up Python HTTP Server:

!\[\[19\_Object (HTB)\_image0012.png]]

&#x20;

&#x20;

Listener port:

{width="3.6041666666666665in" height="0.9166666666666666in"}

&#x20;

This still does not work as it cannot reach my machine.

!\[\[19\_Object (HTB)\_image0014.png]]

&#x20;

There might be some firewalls that prevent this, and as such, we can instead enumerate the Jenkins instance here.

!\[\[19\_Object (HTB)\_image0015.png]]

&#x20;

This would seem to be the Jenkins Home.

!\[\[19\_Object (HTB)\_image0016.png]]

&#x20;

We can view the console output to see what's up.

!\[\[19\_Object (HTB)\_image0017.png]]

&#x20;

Viewing directory the entire jenkins directory:

!\[\[19\_Object (HTB)\_image0018.png]]

&#x20;

Config.xmls:

!\[\[19\_Object (HTB)\_image0019.png]]

&#x20;

!\[\[19\_Object (HTB)\_image0020.png]]

&#x20;

I want to see the admin's one.

!\[\[19\_Object (HTB)\_image0021.png]]

&#x20;

&#x20;

!\[\[19\_Object (HTB)\_image0022.png]]

&#x20;

Interesting, as this hash is encrypted with a master.key and also something called a hudon.util.secret.

!\[\[19\_Object (HTB)\_image0023.png]]

&#x20;

Getting them out:

!\[\[19\_Object (HTB)\_image0024.png]]

&#x20;

{width="14.177083333333334in" height="4.46875in"}

&#x20;

We can save the key, however the hudson.util.secret is something that is rather hard to.

We can get it out using some powershell and base64.

{width="15.010416666666666in" height="1.96875in"}

&#x20;

{width="16.0625in" height="3.4166666666666665in"}

&#x20;

!\[\[19\_Object (HTB)\_image0028.png]]

&#x20;

{width="9.822916666666666in" height="1.4375in"}

Saving config.xml:

{width="13.479166666666666in" height="4.052083333333333in"}

&#x20;

&#x20;

!\[\[19\_Object (HTB)\_image0031.png]]

&#x20;

After which, we can decode this easily using this repository:

[https://github.com/hoto/jenkins-credentials-decryptor](https://github.com/hoto/jenkins-credentials-decryptor)

{width="9.125in" height="2.0104166666666665in"}

&#x20;

!\[\[19\_Object (HTB)\_image0033.png]]

&#x20;

Flag:

!\[\[19\_Object (HTB)\_image0034.png]]

1a81d2ff5945644b7332e1c18b5c6798

&#x20;

PE:

Firstly, let's enumerate the user:

!\[\[19\_Object (HTB)\_image0035.png]]

&#x20;

!\[\[19\_Object (HTB)\_image0036.png]]

&#x20;

!\[\[19\_Object (HTB)\_image0037.png]]

&#x20;

Now, remember that we have a firewall on our hands here and we cannot transfer any files over from our machine.

We would need to configure the firewall to allow us to do stuff.

&#x20;

Bloodhound:

There was no conceivable way forward, but since we had credentials, we could use bloodhound-python to download stuff.

{width="5.177083333333333in" height="0.71875in"}

&#x20;

!\[\[19\_Object (HTB)\_image0039.png]]

&#x20;

!\[\[19\_Object (HTB)\_image0040.png]]

&#x20;

!\[\[19\_Object (HTB)\_image0041.png]]

&#x20;

{width="5.34375in" height="2.90625in"}

&#x20;

Starting BloodHound:

{width="3.96875in" height="1.9583333333333333in"}

&#x20;

Uploading data:

!\[\[19\_Object (HTB)\_image0044.png]]

&#x20;

Take note that we would have to downgrade to Bloodhound 3 because SharpHound is also version 3. Bloodhound version 4 does not work with SharpHound 3.

&#x20;

Bloodhound analysis:

!\[\[19\_Object (HTB)\_image0045.png]]

&#x20;

!\[\[19\_Object (HTB)\_image0046.png]]

&#x20;

So we can privilege escalate to smith using this ForceChangePassword property.

We can change the password of Smith to what we want.

&#x20;

Changing password using PowerView.

!\[\[19\_Object (HTB)\_image0047.png]]

&#x20;

!\[\[19\_Object (HTB)\_image0048.png]]

&#x20;

PE2:

!\[\[19\_Object (HTB)\_image0049.png]]

&#x20;

We can change this user's password as well.

!\[\[19\_Object (HTB)\_image0050.png]]

&#x20;

This is technically very specific Kerberoasting since we are requesting for tickets.

We can first set a SPN for the user maria.

&#x20;

Then we can request for tickets for the user with that SPN. This would be targeted Kerberoasting.

However, PowerView works weirdly for this machine. We can just do a reset to get a good run.

!\[\[19\_Object (HTB)\_image0051.png]]

&#x20;

{width="10.354166666666666in" height="3.7916666666666665in"}

&#x20;

!\[\[19\_Object (HTB)\_image0053.png]]

&#x20;

However, this does not work...

&#x20;

Changing Password:

Using GenericWrite, we can instead change the password of the user instead of Kerberoasting.

However, this cannot work as well.

&#x20;

The only way we can proceed is if we change the logon script that the user uses. This would cause a script to execute everytime he logs in to the device.

However, there isn't much we can do with this knowledge...

This is because the firewall exists and blocks all connections outbound. This would mean we cannot gain any reverse shell.

&#x20;

As such, one of the only things one can do is just view maria's directory or what the user can access.

!\[\[19\_Object (HTB)\_image0054.png]]

&#x20;

Maria has this privilege over the Domain Admins group, meaning that effectively, maria can add her own admins.

In this case, we can create some kind of script that would add users we want to the Domain Admins group.

&#x20;

However, looking at a hint, this is not the intended solution.

!\[\[19\_Object (HTB)\_image0055.png]]

&#x20;

We can create a folder with this to enumerate maria's directory.

&#x20;

Then we set the logon script to be this:

!\[\[19\_Object (HTB)\_image0056.png]]

&#x20;

!\[\[19\_Object (HTB)\_image0057.png]]

&#x20;

We can view the documents folder, which has an odd timestamp.

!\[\[19\_Object (HTB)\_image0058.png]]

&#x20;

!\[\[19\_Object (HTB)\_image0059.png]]

&#x20;

Nothing there...

&#x20;

Desktop:

!\[\[19\_Object (HTB)\_image0060.png]]

&#x20;

!\[\[19\_Object (HTB)\_image0061.png]]

&#x20;

There's this weird folder here, and we can copy it out.

!\[\[19\_Object (HTB)\_image0062.png]]

&#x20;

!\[\[19\_Object (HTB)\_image0063.png]]

&#x20;

Then, we can download that file.

!\[\[19\_Object (HTB)\_image0064.png]]

&#x20;

Libreoffce:

{width="4.25in" height="0.7916666666666666in"}

&#x20;

&#x20;

!\[\[19\_Object (HTB)\_image0066.png]]

&#x20;

There are passwords and stuff too.

!\[\[19\_Object (HTB)\_image0067.png]]

We can try each password.

&#x20;

!\[\[19\_Object (HTB)\_image0068.png]]

&#x20;

Now we are maria, and we already know that maria has writeowner privileges over the Domain Admins group.

We can make ourselves owner of this entire group.

!\[\[19\_Object (HTB)\_image0069.png]]

&#x20;

Give maria all rights:

!\[\[19\_Object (HTB)\_image0070.png]]

&#x20;

!\[\[19\_Object (HTB)\_image0071.png]]

&#x20;

Add maria to domain admins.

!\[\[19\_Object (HTB)\_image0072.png]]

&#x20;

We can relogon.

!\[\[19\_Object (HTB)\_image0073.png]]

&#x20;

!\[\[19\_Object (HTB)\_image0074.png]]

&#x20;

Then we have all admin privileges.

&#x20;

Flag:

!\[\[19\_Object (HTB)\_image0075.png]]

2a5db14b46cd9d7cd1b29a56d530313d

&#x20;
