# Heist

Heist

Monday, 28 March 2022

9:04 pm

A Windows machine with these ports open:

&#x20;

!\[\[09\_Heist\_image001.png]]

&#x20;

!\[\[09\_Heist\_image002.png]]

&#x20;

Port 80 is just some login page.

!\[\[09\_Heist\_image003.png]]

Tried the usual ports, but nothing worked. At least we know it's a PHP server.

&#x20;

I logged in as a guest, and saw this.

!\[\[09\_Heist\_image004.png]]

&#x20;

There's an attachment there and it had credentials within it.

!\[\[09\_Heist\_image005.png]]

&#x20;

Looks like some kind of router configurations with BGP and stuff.

&#x20;

Enable secret is on, so all the passwords are encrypted within the configurations.

Tried to crack the hash there, as it was just md5.

!\[\[09\_Heist\_image006.png]]

&#x20;

Seems that we have a password here. Now, we just need to think about where we can use this password.

Tried to evil-winrm in, but to no avail.

We know the user is called hazard and his password.

&#x20;

Next, we know that the next two passwords are level 7 passwords from Cisco. There are online tools to crack them.

!\[\[09\_Heist\_image007.png]]

These are the passwords, and the lamer one is for rout3r.

&#x20;

Now we have some passwords, and some users.

!\[\[09\_Heist\_image008.png]]

&#x20;

Compile them into one file. We can crackmapexec this because port 445 is open.

&#x20;

{width="9.416666666666666in" height="0.625in"}

&#x20;

Great.

{width="10.354166666666666in" height="1.90625in"}

This does not mean anything, as we cannot even write to IPC$ hence there's no eternalblue being used here.

&#x20;

&#x20;

!\[\[09\_Heist\_image0011.png]]

We can't even connect either.

&#x20;

There's still port 135, which is for RPC.

I ran enum4linux to check it out for me.

&#x20;

It did return some stuff however.

!\[\[09\_Heist\_image0012.png]]

There were some users that were recovered.

&#x20;

Rpcclient could roughly tell me that the users did exist.

!\[\[09\_Heist\_image0013.png]]

But I did not know their names or whatever.

&#x20;

!\[\[09\_Heist\_image0014.png]]

I could only confirm that hazard was there, but I did not know the administrator name. I know the domain too:

&#x20;

!\[\[09\_Heist\_image0015.png]]

&#x20;

This was from the earlier crackmapexec. This is called SUPPORTDESK.'

!\[\[09\_Heist\_image0016.png]]

I was able to find that Hazard was part of this group. We got to brute force the SIDs somehow.

&#x20;

I ran the MSF module that would allow me to.

&#x20;

!\[\[09\_Heist\_image0017.png]]

Found some users to use!

I added them to my passwd file and then did another crackmapexec. Made sure to remove hazard from that list too.

!\[\[09\_Heist\_image0018.png]]

Right, we have another set of creds. Let's try the evil-winrm now.

!\[\[09\_Heist\_image0019.png]]

Right, now we can grab the user flag.

&#x20;

From here, we can see a todo.txt in the same directory as the flag.

&#x20;

!\[\[09\_Heist\_image0020.png]]

&#x20;

!\[\[09\_Heist\_image0021.png]]

Whatever that means.

&#x20;

We seem to have no privileges whatsoever, and there's no service accounts to go to either. Just Hazard, which has no forms of privileges. We also cannot view the wwwroot of the web server to find more things.

&#x20;

!\[\[09\_Heist\_image0022.png]]

&#x20;

!\[\[09\_Heist\_image0023.png]]

&#x20;

That firefox thing looks pretty good, and I wanted to take a look at it.

Transferred it to my machine using smb.

&#x20;

!\[\[09\_Heist\_image0024.png]]

Looking at this, it looks like a password already.

!\[\[09\_Heist\_image0025.png]]

When decrypted using that tool, we can see the string.

&#x20;

Tried to pass the hash for all of them. Didn't work. I think we need to directly dump the password from the memory of the machine.

Went on hacktricks and it pointed me towards procdump, which would dump out the whole memory of everything in clear text.

&#x20;

&#x20;

!\[\[09\_Heist\_image0026.png]]

&#x20;

Went with one of them, and then analysed it from my machine.

Using a the search function, I found this little string here.

&#x20;

!\[\[09\_Heist\_image0027.png]]

Using this I was able to evil-winrm in.

&#x20;

!\[\[09\_Heist\_image0028.png]]

Done.

&#x20;

&#x20;

&#x20;
