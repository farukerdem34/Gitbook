# Clyde

Clyde

Wednesday, 23 March 2022

2:00 pm

A Linux machine with these ports:

!\[\[35\_Clyde\_image001.png]]

For this, I can access FTP! Great.

!\[\[35\_Clyde\_image002.png]]

&#x20;

Within the FTP directory, we can actually access the whole system except for a few parts.

&#x20;

Within the /var/www/html/php4dvd/config/config.php file, I found this.

{width="6.989583333333333in" height="3.34375in"}

This would mean that on the default web port, there is a php4dvd directory and I was right. The default credentials of admin:admin work here.

!\[\[35\_Clyde\_image004.png]]

&#x20;

This does not give me much, so I head to erlang.

This just says it has a port open on port 65000.

&#x20;

We move on to port 15672, which has a login page:

!\[\[35\_Clyde\_image005.png]]

&#x20;

The default credentials are guest and guest, and it gives me this error when trying to log in.

&#x20;

When viewing the requests, we can actually leak the authorization cookie sort of.

!\[\[35\_Clyde\_image006.png]]

This is just basic base64, and we can probably use this.

&#x20;

From the cookie, we can identify it serialises the data we input as guest:guest.

&#x20;

This would mean that we can try the erl cookie RCE trick.

&#x20;

From here, we just need to find a cookie file somewhere. There are exploits related to this, but finding the cookie within FTP is a bit challenging.

!\[\[35\_Clyde\_image007.png]]

Found it!

&#x20;

Now, we just need to get an RCE script going to test this.

We should be using port 65000.

!\[\[35\_Clyde\_image008.png]]

&#x20;

!\[\[35\_Clyde\_image009.png]]

&#x20;

!\[\[35\_Clyde\_image0010.png]]

Within that, we can sort of see our reply. Let's try with a different command like a ping.

&#x20;

{width="9.8125in" height="4.708333333333333in"}

This works!

&#x20;

Now, we have RCE and can get onto the machine.

&#x20;

!\[\[35\_Clyde\_image0012.png]]

&#x20;

!\[\[35\_Clyde\_image0013.png]]

Great.

&#x20;

!\[\[35\_Clyde\_image0014.png]]

&#x20;

There's one other user called dana here, and we seem to be this weird user.

Remember, we still have that other password we saw just now.

&#x20;

&#x20;

!\[\[35\_Clyde\_image0015.png]]

Nmap can be run on this!

&#x20;

!\[\[35\_Clyde\_image0016.png]]

&#x20;

!\[\[35\_Clyde\_image0017.png]]

&#x20;

!\[\[35\_Clyde\_image0018.png]]

Ezpz.

&#x20;

&#x20;
