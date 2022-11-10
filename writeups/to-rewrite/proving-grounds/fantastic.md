# Fantastic

Fantastic

Wednesday, 23 March 2022

3:49 pm

A linux machine with these ports:

!\[\[36\_Fantastic\_image001.png]]

Port 3000 is a login page for Grafana and 9090 is just some GUI for making graphs.

&#x20;

The website on port 3000 looks like this:

!\[\[36\_Fantastic\_image002.png]]

There is a good exploit for this version which allows for reading of the files within the web server.

!\[\[36\_Fantastic\_image003.png]]

&#x20;

!\[\[36\_Fantastic\_image004.png]]

This works well. We need some password for this machine, so let's view the .conf file for grafana.

!\[\[36\_Fantastic\_image005.png]]

&#x20;

This is the only user on the machine.

&#x20;

This would be in the /etc/grafana/grafana.ini file.

!\[\[36\_Fantastic\_image006.png]]

&#x20;

There seems to be a root user on this, but not confirmed yet.

&#x20;

!\[\[36\_Fantastic\_image007.png]]

&#x20;

!\[\[36\_Fantastic\_image008.png]]

We can read the /var/lib/grafana/grafana.db to read the password file and get out the administrator hash.

&#x20;

!\[\[36\_Fantastic\_image009.png]]

&#x20;

From here, we can grab the password and actually decode it.

!\[\[36\_Fantastic\_image0010.png]]

&#x20;

When looking through the database, I also saw this thing:

!\[\[36\_Fantastic\_image0011.png]]

This was a password for sysadmin or something. It was clearly AES encrypted. Now, we need to find some key.

&#x20;

!\[\[36\_Fantastic\_image0012.png]]

This should do. There are methods to decrypt this thing, and one such method is using [this](https://github.com/jas502n/Grafana-CVE-2021-43798).

&#x20;

!\[\[36\_Fantastic\_image0013.png]]

We got a password from this.

&#x20;

Using this, we can SSH in as sysadmin!

!\[\[36\_Fantastic\_image0014.png]]

&#x20;

When checking the id of this user, we seem to be part of the disk group.

This is a clear vector for PE.

!\[\[36\_Fantastic\_image0015.png]]

&#x20;

From here, we can check of all the stuff that we have from here.

!\[\[36\_Fantastic\_image0016.png]]

&#x20;

Now, with disk group, we can almost be root but not really.

!\[\[36\_Fantastic\_image0017.png]]

&#x20;

We can read the root key though!

!\[\[36\_Fantastic\_image0018.png]]

&#x20;

!\[\[36\_Fantastic\_image0019.png]]

&#x20;

Pretty fun box looking through stuff for passwords.

&#x20;

&#x20;
