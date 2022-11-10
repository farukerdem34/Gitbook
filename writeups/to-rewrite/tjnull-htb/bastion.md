# Bastion

Bastion

Sunday, 13 February 2022

9:16 pm

This is a Windows machine.

The target IP is 10.10.10.134.

My IP is 10.10.16.9.

&#x20;

The machine has the usual SMB ports that are open, as well as SSH

!\[\[36\_Bastion\_image001.png]]

&#x20;

There are actually more ports that are open on the machine. We should do a full port scan.

!\[\[36\_Bastion\_image002.png]]

&#x20;

Better. Anyways doing some googling reveals that port 5985 is actually winRM running, and it uses HTTP as a transport method.

This can be enumerated further using tools like evil-winrm.

&#x20;

Anyways I tried enumerating the SMB shares more using SMBclient.

&#x20;

!\[\[36\_Bastion\_image003.png]]

Seems to have some files we can connect to.

&#x20;

Let's try the backups file, since it's not a default file.

When connecting to it, we can find a note.txt.

!\[\[36\_Bastion\_image004.png]]

When we get it, there's that message, which I don't know what It means. I just know there's a VPN to another office, and that means there's probably some other network or IP that is up?

Anyways delving further into the file, we know it's a backup, and there's probably some form of passwords on it there.

&#x20;

&#x20;

&#x20;

I downloaded everything that I could from here. I know this is a Windows Backup, but how do I go about it? There has to be some kind of tool that can view this properly. After a bit of research, I found this on Google.

!\[\[36\_Bastion\_image005.png]]

&#x20;

Could give it a try, looks pretty legit.

!\[\[36\_Bastion\_image006.png]]

&#x20;

We do indeed have a vhd file that we could use.

Found this command online.

&#x20;

&#x20;

!\[\[36\_Bastion\_image007.png]]

&#x20;

Followed it exactly, and this dumped out the stuff that was held within the Windows Backup file, which was really useful.

!\[\[36\_Bastion\_image008.png]]

This basically gave us everything we needed.

!\[\[36\_Bastion\_image009.png]]

&#x20;

From here, I need to find some kind of password storage or something.

I know that passwords are stored within the SAM file or something.

!\[\[36\_Bastion\_image0010.png]]

I suppose it's here somewhere. A bit of googling reveals that we can use samdump2 to dump out the contents of the file in a readable format.

&#x20;

From here, we can do this.

!\[\[36\_Bastion\_image0011.png]]

Now we can crack this shit.

Windows stores passwords as NTLM hashes, which can be brute forced.

&#x20;

{width="13.947916666666666in" height="3.1979166666666665in"}

&#x20;

!\[\[36\_Bastion\_image0013.png]]

We have that password, and we can finally SSH in!

&#x20;

It works out well and we can get the user flag.

&#x20;

{width="5.59375in" height="5.03125in"}

Solid, now let's try to privilege escalate.

&#x20;

Interestingly, there is a WAF on the systeminfo function, whereby we cannot call it.

&#x20;

!\[\[36\_Bastion\_image0015.png]]

I followed a [Windows Privilege Escalation checklist](https://www.fuzzysecurity.com/tutorials/16.html) to enumerate everything systematically. I highly recommend this to anyone that is interested.

&#x20;

Anyways I went to the user directory and did a dir /all and investigated every single file.

&#x20;

{width="12.072916666666666in" height="3.65625in"}

&#x20;

First file up is AppData\Roaming.

Inside this file is a mRemoteNG file.

!\[\[36\_Bastion\_image0017.png]]

This is a remote access connections manager for Windows.

&#x20;

There's a log file, and we can start viewing more information that is useful.

{width="10.135416666666666in" height="1.03125in"}

&#x20;

Interestingly, there's one confCons.xml, and upon viewing it, we are able to see one line of interesting text.

!\[\[36\_Bastion\_image0019.png]]

&#x20;

I thought it was base64, but it was not. I wanted to figure out what kind of password does this mRemoteNG application use.

In this case, after looking around on Github, I found [this repository](https://github.com/haseebT/mRemoteNG-Decrypt) that has what I needed.

&#x20;

Cool!

Clone that shit into our directory.

&#x20;

{width="7.28125in" height="2.875in"}

&#x20;

Use it and we should get a password.

{width="12.3125in" height="0.7604166666666666in"}

&#x20;

Cool, now let's try to SSH in.

And we are in.

&#x20;

{width="6.270833333333333in" height="1.25in"}

&#x20;

!\[\[36\_Bastion\_image0023.png]]

This was a really, really fun box. I really enjoyed looking through the backups to find the files I needed. That was really refreshing. The usage of dir /all also helped me out a great deal as well

&#x20;
