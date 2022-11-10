# Nappa

Nappa

Wednesday, June 15, 2022

11:58 PM

Interesting Linux machine.

&#x20;

This involved a lot of reading from the HTML Page Source to find out the password of the administrator user.

&#x20;

Next, we needed to read page source again to find out some interesting comments left behind by the user, allowing us to do RCE when changing the HTTP verb used and also changing parameters being sent to the user.

&#x20;

After using this RCE to gain a shell, we would need to see the abnormally large .bashrc file to find a base32-encoded root SSH Key.

&#x20;

Not realistic, but I'll take it.

&#x20;

Anyways, this was the box that I did over a holiday, so there's no writeup as usual.

&#x20;

&#x20;

&#x20;
