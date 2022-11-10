# Cascade (HTB)

This is a Windows AD machine.

The Machine IP is 10.10.10.182.

My IP is 10.10.16.9.

!\[\[10\_Cascade (HTB)\_image001.png]]

!\[\[10\_Cascade (HTB)\_image002.png]]   Enum4linux returned lots of things. !\[\[10\_Cascade (HTB)\_image003.png]]

&#x20;!\[\[10\_Cascade (HTB)\_image004.png]]

Put the users into a user file. Cascade.local is the domain here. Tried some crackmepexec to determine some SMB credentials, but that didn't work.

Ran an LDAP search just to make sure I had everything.

!\[\[10\_Cascade (HTB)\_image005.png]] While looking at the users, I found this weird string:

!\[\[10\_Cascade (HTB)\_image006.png]] Seems that our user r.thompson had this.

Decoded, this gave a password: !\[\[10\_Cascade (HTB)\_image007.png]]

Anyways, I did some checking with the credentials. !\[\[10\_Cascade (HTB)\_image008.png]]

There's this Data share that was out of place.

Logged onto it and found some directories. !\[\[10\_Cascade (HTB)\_image009.png]]

Right, so every directory except for one blocked me.

Looked up a quick method to get everything I could into my machine, and found this on Github: !\[\[10\_Cascade (HTB)\_image010.png]]

Got 4 files from this:

!\[\[10\_Cascade (HTB)\_image011.png]]

Within Email Archives, there was this HTML file, and when viewed:

!\[\[10\_Cascade (HTB)\_image012.png]]

Right, so this gave us a password to use.

Within the bin log, there was this:

!\[\[10\_Cascade (HTB)\_image013.png]]

The VNC reg gave me another password.

!\[\[10\_Cascade (HTB)\_image014.png]]

In hex this time. Gave me this: !\[\[10\_Cascade (HTB)\_image015.png]] I have no idea what's that, but anyways. So this password is encrypted somehow, and can actually decrypt this. While googling, I came across this command: !\[\[10\_Cascade (HTB)\_image016.png]] Used the hex there and got this:

!\[\[10\_Cascade (HTB)\_image017.png]] Right, so now we have this password to some user.

This was inside the user file called s.smith, so why not try some evil-winRM:

!\[\[10\_Cascade (HTB)\_image018.png]]&#x20;

Great! Grab the user flag from there.

Now, we just need to find a way to privilege escalate.

There was another user, presumably the arksvc one.

!\[\[10\_Cascade (HTB)\_image019.png]] Perhaps this has got to do with the TempAdmin user they were talking about.

I ran a winpeas on the machine, transferring over HTTP. !\[\[10\_Cascade (HTB)\_image020.png]]

Didn't get much from that. When running a net user to check local group memberships, I found this: !\[\[10\_Cascade (HTB)\_image021.png]]

Audit share?

!\[\[10\_Cascade (HTB)\_image022.png]]

There seems to be a hint towards using SMB again, so why not? !\[\[10\_Cascade (HTB)\_image023.png]]

Right, so let's investigate that Audit$ thing on the Windows machine.

!\[\[10\_Cascade (HTB)\_image024.png]] There's one DB folder. Within it, there's an audit.db file.

!\[\[10\_Cascade (HTB)\_image025.png]]

Right, so let's just copy all the files into my Linux machine again.

!\[\[10\_Cascade (HTB)\_image026.png]]

Right, sqlite3 again.

!\[\[10\_Cascade (HTB)\_image027.png]]

This was all of the data:

!\[\[10\_Cascade (HTB)\_image028.png]]

There's that base64 looking string again.

However, this is not base64... This is some net fiddle stuff, and googling the exact string gives us this: !\[\[10\_Cascade (HTB)\_image029.png]] This is the decrypted version of the string.

Anyways, evil-winrm into the machine again, this time as ArkSvc.

!\[\[10\_Cascade (HTB)\_image030.png]] Now, we have a few more things to look at, especially this Recycle Bin. Went on Hacktricks and found this neat oneliner. Turns out we can read deleted AD objects within this machine. Again, we see the TempAdmin user!

!\[\[10\_Cascade (HTB)\_image031.png]] There's yet another password there. And within the HTML file we read earlier, this had the same password as the admin account.

When decoded, this gave us the password!

!\[\[10\_Cascade (HTB)\_image032.png]]

!\[\[10\_Cascade (HTB)\_image033.png]]

&#x20;
