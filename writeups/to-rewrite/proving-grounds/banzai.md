# Banzai

Banzai

Saturday, 19 March 2022

8:08 pm

A Linux machine with these ports:

&#x20;

!\[\[21\_Banzai\_image001.png]]

Port 8080 has nothing, and port 25 gives us a DNS name to consider.

&#x20;

Right at port 8295, there's this team which we might need to take note of:

!\[\[21\_Banzai\_image002.png]]

Gobuster indicates this is a php website.

&#x20;

There's one weakness, in which is the FTP website. The credentials for it are admin:admin.

For some reason, I was not able to see the files within it. So I just put some cmd.php file there and prayed.

!\[\[21\_Banzai\_image003.png]]

It would just get stuck here, and that's quite annoying sometimes.

Box stopped working, will come back to it later.

&#x20;

Eventually, this works.

!\[\[21\_Banzai\_image004.png]]

&#x20;

We can do a nc reverse shell back to us and get in.

From there, we can find some config.php which contains some credentials for a database.

&#x20;

Seems that root is running the database, and we can do some stuff with that.

&#x20;

!\[\[21\_Banzai\_image005.png]]

&#x20;

!\[\[21\_Banzai\_image006.png]]

From here, we can follow instructions from [here](https://medium.com/r3d-buck3t/privilege-escalation-with-mysql-user-defined-functions-996ef7d5ceaf) in order to execute commands as root.

&#x20;

From there, we can edit the /etc/passwd file easily after giving permissions to all to do it.

&#x20;

!\[\[21\_Banzai\_image007.png]]

&#x20;

!\[\[21\_Banzai\_image008.png]]

w
