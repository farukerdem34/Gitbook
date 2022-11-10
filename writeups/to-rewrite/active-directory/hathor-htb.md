# Hathor (HTB)

Hathor (HTB)

Friday, July 1, 2022

11:38 AM

This is one insane AD machine here...Crazy difficult and crazy long but crazy fun.

Nmap scan:

!\[\[21\_Hathor (HTB)\_image001.png]]

&#x20;

There's a port 80, which I want to investigate first.

!\[\[21\_Hathor (HTB)\_image002.png]]

&#x20;

From here, we can actually log in to this domain.

I used admin@admin.com:admin to log in.

&#x20;

!\[\[21\_Hathor (HTB)\_image003.png]]

&#x20;

Interestingly, there's this filemanager, which I think can be used to upload some form of shell or something.

!\[\[21\_Hathor (HTB)\_image004.png]]

&#x20;

Since this was a Windows machine, and also an IIS server:

!\[\[21\_Hathor (HTB)\_image005.png]]

&#x20;

We also know that this is running mojoportal, which is a CMS that has the default credentials I used earlier.

We should be using some form of aspx shells.

Edit the htm file with the aspx shell, and we cannot change the extension through editing, but we can copy files that are aspx.

!\[\[21\_Hathor (HTB)\_image006.png]]

&#x20;

Now after copying this, it does not seem to appear within the file manager anymore.

I'm fairly confident that it's somewhere on this page.

&#x20;

After a bit of fuzzing, I determined that it had to be somewhere with /media/logos/fragment1.aspx.

[http://10.129.165.35/Data/Sites/1/media/logos/fragment1.aspx](http://10.129.165.35/Data/Sites/1/media/logos/fragment1.aspx)

&#x20;

This works.

Curled this and got a shell.

{width="7.59375in" height="3.125in"}

&#x20;

Now, we are not allowed to do anything on this user.

&#x20;

Within the C:\ directory, there was this Get-ADPassowrds file.

!\[\[21\_Hathor (HTB)\_image008.png]]

&#x20;

Seems that this is a hint to use the powershell script to receive something.

I ran the script to see what would happen.

&#x20;

Nothing happened.

Anyways, this directory has an identical one in Github. We can compare the two to see which directories are created by the machine and not there by default.

&#x20;

!\[\[21\_Hathor (HTB)\_image009.png]]

&#x20;

The whole CSV thing is new, and we can take a look at some of them.

&#x20;

!\[\[21\_Hathor (HTB)\_image0010.png]]

&#x20;

We can find a username and a hash for this user.

!\[\[21\_Hathor (HTB)\_image0011.png]]

&#x20;

So we have credentials, but this cannot be used to authenticate to SMB, instead we can test with LDAP or something.

Bloodhound does not work.

&#x20;

However, when using ldapsearch, we can find a lot of information.

{width="9.9375in" height="1.0in"}

&#x20;

But there's nothing much that we can get from this.

The password and username has to be used somehow, but I'm not sure how.

&#x20;

There are shares on the device.

!\[\[21\_Hathor (HTB)\_image0013.png]]

&#x20;

This is clearly a share pointing to C:\share on the main directory. There should be a way to mount or connect and see what files are inside of it.

Since we have some form of credentials, we can try Powershell remoting, which now doesn't work.

&#x20;

Now, we can try to access this share using the credentials that we have.

&#x20;

!\[\[21\_Hathor (HTB)\_image0014.png]]

&#x20;

Then we can view the scripts and stuff.

!\[\[21\_Hathor (HTB)\_image0015.png]]

&#x20;

We can see how there is DLL here.

We can grab an impersonation shell for ASPX.

{width="7.3125in" height="2.5104166666666665in"}

&#x20;

Then we can connect here.

Now we can view the share for whatever information is there.

&#x20;

WIP!

&#x20;

&#x20;
