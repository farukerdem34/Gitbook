# Synapse

Synapse

Wednesday, July 13, 2022

10:55 AM

Nmap scan:

!\[\[74\_Synapse\_image001.png]]

&#x20;

!\[\[74\_Synapse\_image002.png]]

&#x20;

!\[\[74\_Synapse\_image003.png]]

&#x20;

Seems that signing is disabled.

&#x20;

SMB:

{width="3.9479166666666665in" height="0.53125in"}

&#x20;

{width="7.78125in" height="2.1666666666666665in"}

&#x20;

!\[\[74\_Synapse\_image006.png]]

&#x20;

{width="9.8125in" height="2.1875in"}

&#x20;

Likely that we would need to gain some credentials from the website or something, then try it with SMB and also FTP. There are no writeable shares, hence we can't get any form of shells.

&#x20;

Port 80:

!\[\[74\_Synapse\_image008.png]]

&#x20;

There's a file manager and Login I'm interested in.

&#x20;

Login:

Doesn't work, relies fully on JS and there is nothing transmitted network wise.

&#x20;

File manager:

!\[\[74\_Synapse\_image009.png]]

&#x20;

elFinder has some exploits going for it.

&#x20;

We can see that this is a vulnerable version.

!\[\[74\_Synapse\_image0010.png]]

&#x20;

However, we need credentials in order to use this manager, because it seems to be restricted to administrators only.

&#x20;

Users:

!\[\[74\_Synapse\_image0011.png]]

We can find out more about the domain, and that this user is called mindsflee.

&#x20;

There are some buttons down here.

Most of them lead to an under construction page.

&#x20;

One of them allows for editing of our avatar.

!\[\[74\_Synapse\_image0012.png]]

We can try to upload some form of shell, but it seems that it checks on what type of file is being uploaded.

!\[\[74\_Synapse\_image0013.png]]

&#x20;

!\[\[74\_Synapse\_image0014.png]]

&#x20;

This shtml file seems weird, because it checks for SSI according to the server. This is perhaps vulnerable to forms of SSI attacks, which basically means that we can manipulat ethe file system and execute shell commands.

&#x20;

The SSI directives are injected in input fields and sent to the server, and in this case, it reads our IP address, and if we can find a way to spoof that, it would be a lot easier.

&#x20;

The other parameter that we control is the name of the file that is being uploaded. We can attempt to inject stuff into the name as per the HTML.

!\[\[74\_Synapse\_image0015.png]]

&#x20;

!\[\[74\_Synapse\_image0016.png]]

&#x20;

This sort of works.

We can see that it tries to load the file name as some kind of HTML.

&#x20;

We can achieve RCE in this manner by uploading an edited file name.

!\[\[74\_Synapse\_image0017.png]]

&#x20;

This works, and then we load the request in Burpsuite to achieve SSI RCE.

!\[\[74\_Synapse\_image0018.png]]

&#x20;

We can see there is the vsftpd.conf file there, which we can view.

!\[\[74\_Synapse\_image0019.png]]

&#x20;

Next, we can gain a shell.

From here, we can view the hidden directory as well.

!\[\[74\_Synapse\_image0020.png]]

&#x20;

!\[\[74\_Synapse\_image0021.png]]

&#x20;

We should be able to upload some sort of shell to gain RCE through a proper php shell.

&#x20;

We can also read the smb.conf.

&#x20;

There are very limited commands that we can do with this shell, so as a start we should try to upload some kind of jpg file with a shell embedded within it.

!\[\[74\_Synapse\_image0022.png]]

&#x20;

We can gain a shell using this.

!\[\[74\_Synapse\_image0023.png]]

&#x20;

{width="7.125in" height="1.78125in"}

&#x20;

Flag:

!\[\[74\_Synapse\_image0025.png]]

&#x20;

PE:

I ran a LINPEAS to check on possible vectors. We should be aiming to become the user next.

Linpeas didn't do much.

&#x20;

In the home directory of the user, there is a script called synapse commander that takes arguments and does stuff.

!\[\[74\_Synapse\_image0026.png]]

&#x20;

When running this script, it takes an input but doesn't do much with it.

It seems to run a check for assembly files.

&#x20;

Not sure what this one does, but we move on for now.

&#x20;

Within the home directory of this user, there are other files of interest.

!\[\[74\_Synapse\_image0027.png]]

&#x20;

Within gnupg, there are creds.priv and creds.txt.gpg that is encoded.

!\[\[74\_Synapse\_image0028.png]]

&#x20;

We can use GPG to decrypt this.

&#x20;

!\[\[74\_Synapse\_image0029.png]]

&#x20;

When trying to decode this, it says we are unable to do so because we are denied permission.

In this case, we would have to think of something else.

&#x20;

I transferred everytning to my machine, and saw that it required a passphrase to import this key.

!\[\[74\_Synapse\_image0030.png]]

&#x20;

We can gpg2john this.

{width="4.15625in" height="1.125in"}

&#x20;

{width="9.8125in" height="3.3541666666666665in"}

&#x20;

Then we can import this key.

{width="5.395833333333333in" height="2.0104166666666665in"}

&#x20;

{width="5.9375in" height="2.1041666666666665in"}

&#x20;

Then we can SSH in as the user.

&#x20;

PE2:

Then we find ourselves back to the original script.

{width="8.65625in" height="1.8020833333333333in"}

&#x20;

When we run this with sudo, we can essentially see that this creates some kind of file within the /tmp directory that is a socket.

!\[\[74\_Synapse\_image0036.png]]

&#x20;

This is socket file.

Basically, we can listen to this socket and then perhaps go ahead and inject code for RCE.

!\[\[74\_Synapse\_image0037.png]]

&#x20;

I tried something like this.

And we can get bash the SUID binary.

!\[\[74\_Synapse\_image0038.png]]

&#x20;

Root shell:

!\[\[74\_Synapse\_image0039.png]]

&#x20;

Flag:

!\[\[74\_Synapse\_image0040.png]]

&#x20;

&#x20;
