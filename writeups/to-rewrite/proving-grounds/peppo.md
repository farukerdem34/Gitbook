# Peppo

Peppo

Saturday, 19 March 2022

8:33 pm

Has these ports:

!\[\[22\_Peppo\_image001.png]]

We can log in to the website with admin:admin.

&#x20;

The website has nothing of interest.

&#x20;

Let's take a look at port 113.

Following some hacktricks, we are able to enumerate some form of users

&#x20;

Ident-user-enum would take each port and identify who's the user for them.

Was unable to find some new ports, hence I ran another nmap just to scan again and make sure I got everything, as sometimes the -p- flag doesn't pick up on everything.

&#x20;

From here, we are able to determine more users:

!\[\[22\_Peppo\_image002.png]]

&#x20;

!\[\[22\_Peppo\_image003.png]]

There's one user called eleanor, and since the password to that website was admin:admin, I tried eleanor as the password and it works.

&#x20;

Within this machine, it was hard to find out the commands as cat and bash don't seem to be installed.

It's a very restricted shell, and we are unable to find out anything from the files.

&#x20;

Checking the path, we seem to take commands from the /home/usr/bin.

!\[\[22\_Peppo\_image004.png]]

&#x20;

Within bin, there were some commands we could use.

!\[\[22\_Peppo\_image005.png]]

&#x20;

Using ed, we can break out of this shell.

&#x20;

!\[\[22\_Peppo\_image006.png]]

&#x20;

Once this is done, we can change the PATH and begin using proper commands. This can be done using the EXPORT PATH= \<path>:$PATH command.

&#x20;

!\[\[22\_Peppo\_image007.png]]

&#x20;

Checking the id, we seem to be part of the docker group.

&#x20;

!\[\[22\_Peppo\_image008.png]]

&#x20;

!\[\[22\_Peppo\_image009.png]]

&#x20;

From here, we just need to find the images that are present, and this can be done using docker image ls.

&#x20;

!\[\[22\_Peppo\_image0010.png]]

&#x20;

!\[\[22\_Peppo\_image0011.png]]

&#x20;

Rooted.

&#x20;

!\[\[22\_Peppo\_image0012.png]]

&#x20;

Simple but tricky machine.
