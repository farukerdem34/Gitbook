# Blackgate

Blackgate

Sunday, June 26, 2022

2:39 PM

Nmap scan:

!\[\[61\_Blackgate\_image001.png]]

&#x20;

!\[\[61\_Blackgate\_image002.png]]

&#x20;

This version of Redis is exploitable, so let's test it out.

&#x20;

RCE:

[https://github.com/n0b0dyCN/redis-rogue-server](https://github.com/n0b0dyCN/redis-rogue-server)

{width="7.34375in" height="4.125in"}

&#x20;

Great, we have RCE in like 3 minutes. From here, we can restart the box and then get it to give us a reverse shell.

{width="8.666666666666666in" height="4.3125in"}

&#x20;

{width="7.125in" height="1.4895833333333333in"}

&#x20;

Flag:

!\[\[61\_Blackgate\_image006.png]]

928198fa4e7d623077a7fd8681f40594

&#x20;

PE:

We have some sudo privileges:

!\[\[61\_Blackgate\_image007.png]]

&#x20;

When running this, we can see that we need some authorization key.

!\[\[61\_Blackgate\_image008.png]]

&#x20;

So we should be trying to find out what does this binary do, and perhaps where we can get this key of sorts.

We can transfer this back to our machine and then strings it.

&#x20;

!\[\[61\_Blackgate\_image009.png]]

&#x20;

Afterwards, it sems to execute systemctl and checks on the status of redis being used here.

{width="7.208333333333333in" height="4.78125in"}

&#x20;

From here, we can easily gain a shell by just doing this:

{width="5.895833333333333in" height="2.1354166666666665in"}

&#x20;

Flag:

!\[\[61\_Blackgate\_image0012.png]]

e27dabe32a17e7ec0ece064b03310ce8
