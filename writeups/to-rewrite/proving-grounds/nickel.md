# Nickel

Nickel

Friday, 18 March 2022

9:56 am

A Windows machine with a weird exploit.

&#x20;

We can retrieve the base64 encoded password after changing a few HTTP request headers easily.

&#x20;

Then SSH in.

!\[\[93\_Nickel\_image001.png]]

From there, transfer some pdf file from the machine to us and brute force decode the password for it.

&#x20;

!\[\[93\_Nickel\_image002.png]]

Send some requests to this [http://nickel/](http://nickel/)?, which when checked outputs ascii code to verify it is the administrator of the system.

&#x20;

From there, since we have the admin cmd, we can just add ariah to administrators.

Port 3389 is open, and hence we can just RDP in as the admin.

&#x20;

!\[\[93\_Nickel\_image003.png]]

&#x20;
