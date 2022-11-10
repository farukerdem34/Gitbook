# Pebbles

Pebbles

Saturday, 19 March 2022

10:13 pm

Interesting Windows machine.

Anonymous FTP login, leading to rar cracking that leads to MS SQL credentials.\
Followed by some simple RCE through MS SQL.

&#x20;

Get a reverse shell, and digging for passwords stored within the registry.

&#x20;

Once that is found, logging into the device using the account jane found with the password.

!\[\[24\_Pebbles\_image001.png]]

Next, we are required to exploit Plantronics.

&#x20;

Follow the MSF exploit:

!\[\[24\_Pebbles\_image002.png]]

Make a file with this content.

&#x20;

Save it within ProgramData\Plantronics\Spokes3G.

&#x20;

Instantly, an admin shell would pop up.

&#x20;

{width="3.7708333333333335in" height="1.0625in"}

&#x20;

!\[\[24\_Pebbles\_image004.png]]

&#x20;
