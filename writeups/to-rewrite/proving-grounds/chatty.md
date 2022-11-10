# Chatty

Chatty

Sunday, 3 April 2022

1:20 pm

This is a Linux machine that has only one service running on it, and that's Rocket.Chat.

&#x20;

This version of rocket chat is vulnerable to the NoSQLI exploit here.

!\[\[50\_Chatty \_image001.png]]

&#x20;

From here, we can grab a token through Blind NoSqli injection and that allows us to compromise something. We just need to find some kind of low-privilege token or something.

&#x20;

Going to the /api/info directory, we can expose the version of Rocket.Chat this is using.

&#x20;

!\[\[50\_Chatty \_image002.png]]

&#x20;

We need to create another user that is present on the server.

!\[\[50\_Chatty \_image003.png]]

&#x20;

This is the administrator account email, and we either need to create another user with the same email, or perhaps we must find a low privileged user on this website.

!\[\[50\_Chatty \_image004.png]]

There is also this user, which seems to be a bot perhaps.

&#x20;

I tried the exploit with my own created user:

!\[\[50\_Chatty \_image005.png]]

This would slowly and painfully get the token out that we need for an RCE on the administrator account. Funnily enough, the writer of this exploit is the creator of the machine, so this means we definitely got the correct exploit here.

However, since we have the freedom of registering users and choosing our own passwords, there's no need to reset the password of the user and change it. From this, we can take out the slow portion of the script that takes the cookie via Blind SQLI.

&#x20;

We would also need to change the password within the code to whatever password we choose with our registered user.

{width="6.84375in" height="3.0625in"}

We can remove this portion and go straight to the administrator token and stuff.

&#x20;

This would also take a while, but definitely halves the script time in half.

Once exploited successfully, we can get a shell here that is able to ping our device.

!\[\[50\_Chatty \_image007.png]]

&#x20;

{width="9.385416666666666in" height="3.1041666666666665in"}

This would indicate we can get a reverse shell from here.

&#x20;

{width="9.25in" height="0.96875in"}

&#x20;

!\[\[50\_Chatty \_image0010.png]]

&#x20;

!\[\[50\_Chatty \_image0011.png]]

From here, I ran a linpeas to check on the other portions of the machine.

Within the SUID portion, there was this part:

!\[\[50\_Chatty \_image0012.png]]

There was an unknown binary.

!\[\[50\_Chatty \_image0013.png]]

This was the version, which was vulnerable to a certain local PE exploit.

!\[\[50\_Chatty \_image0014.png]]

&#x20;

This exploit for writing the stuff does not really work, so I googled and tried to look at another one.

!\[\[50\_Chatty \_image0015.png]]

&#x20;

When downloaded onto the target machine, and run, this was able to run the exploit for us and put a sh file within /var/tmp file.

&#x20;

!\[\[50\_Chatty \_image0016.png]]

When executed, we are root.

&#x20;

!\[\[50\_Chatty \_image0017.png]]

&#x20;

!\[\[50\_Chatty \_image0018.png]]

&#x20;
