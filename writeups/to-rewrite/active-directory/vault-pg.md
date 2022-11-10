# Vault (PG)

Vault (PG)

Thursday, 24 March 2022

10:22 am

A unique AD machine that involves some malicious lnk files triggering a response in the machine and resulting in the capturing of the hash through MITM attempts.

&#x20;

There is one share called DocumentsShare that we can read and write to, and we can place files within there that would have a path to an external SMB server, and when placed there, it would allow for us to be able to capture authentication requests made by the server through setting up a responder server that captures the hash.

&#x20;

Unique attack that I've never done before and definitely something to keep in mind when doing AD.

&#x20;

The attack starts with generating a malicious .lnk file, which is basically a symlink for Windows.

&#x20;

!\[\[13\_Vault (PG) \_image001.png]]

Once this file has been created, we can place it within the share and start responder.

&#x20;

!\[\[13\_Vault (PG) \_image002.png]]

&#x20;

And on responder, we'll receive this

{width="7.90625in" height="1.9895833333333333in"}

&#x20;

This is a hash for the user. Crack it and evil-winrm in.

&#x20;

!\[\[13\_Vault (PG) \_image004.png]]

&#x20;

!\[\[13\_Vault (PG) \_image005.png]]

&#x20;

From here, after a few hours and days, the hint was to take a GPO and begin exploitation using that. I was not familiar with this and will take a break while solving it.

&#x20;

Ran a powerview on this thing to view all of the GPOs.

!\[\[13\_Vault (PG) \_image006.png]]

We need to find some sort of permissions that we can edit here.

&#x20;

!\[\[13\_Vault (PG) \_image007.png]]

&#x20;

This tells us that we can modify this one GPO here.

!\[\[13\_Vault (PG) \_image008.png]]

&#x20;

Looks like we can abuse some of them.

&#x20;

!\[\[13\_Vault (PG) \_image009.png]]

&#x20;

Now we just need to run a gpupdate.

From there, we are now an administrator.

&#x20;

!\[\[13\_Vault (PG) \_image0010.png]]

&#x20;

Great, now we are an administrator.

&#x20;

!\[\[13\_Vault (PG) \_image0011.png]]

Psexec in and we are now root.

&#x20;

!\[\[13\_Vault (PG) \_image0012.png]]

&#x20;
