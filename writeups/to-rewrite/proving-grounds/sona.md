# Sona

Sona

Monday, 21 March 2022

5:06 pm

An interesting Linux machine that seems to block all Nmaps on it, and the description says not to -sV -sC it.

&#x20;

!\[\[26\_Sona\_image001.png]]

&#x20;

These seem to be present on the machine.

!\[\[26\_Sona\_image002.png]]

This is vulnerable to a certain exploit.

&#x20;

!\[\[26\_Sona\_image003.png]]

Interestingly, there's this telent port too.

&#x20;

!\[\[26\_Sona\_image004.png]]

&#x20;

Now, telnet is unencrypted, so we may need to check the wireshark packets for hints too. Anyways the connection seems to cut immediately if we use telnet, so we can use nc instead.

!\[\[26\_Sona\_image005.png]]

Seems that we would need to identify the administrator password before proceeding with the RCE exploit.

&#x20;

From here, we can actually restore the backup and perhaps get the default credentials back to the website.

&#x20;

!\[\[26\_Sona\_image006.png]]

&#x20;

!\[\[26\_Sona\_image007.png]]

Not sure if this is important, but it does tell us the user.

&#x20;

!\[\[26\_Sona\_image008.png]]

I don't think I have to answer these. Let's think, the box tell us to backup, comply and repeat.

&#x20;

While looking through the requests on Burp, there was this one request with loads of hashes.

&#x20;

!\[\[26\_Sona\_image009.png]]

Decrypting the anonymousUsername hash reveals that this is a SHA1 hash. Quite sure this is a rabbit hole as this hash cannot be cracked.

&#x20;

Turns out the Telnet thing was right. All we had to do was just guess blackleo and we would get the admin password.

&#x20;

From there, we can get RCE on this interface to get a shell into it. Once we have achieved a shell, we would just need to PE using a cronjob that imports the base64 python module that we can hijack easily.

&#x20;

Dones.

&#x20;

&#x20;

&#x20;
