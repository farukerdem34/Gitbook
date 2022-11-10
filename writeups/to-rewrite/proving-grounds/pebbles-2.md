# Pebbles (2)

Pebbles

Tuesday, 22 March 2022

10:50 am

The following ports were open:

!\[\[28\_Pebbles\_image001.png]]

&#x20;

!\[\[28\_Pebbles\_image002.png]]

&#x20;

Port 3305 had a zoneminder extension to it. Port 80 was a simple login page, and port 8080 was a tomcat page that had no links working on it, presumably a rabbit hole again.

&#x20;

When doing a searchsploit, it reveals there is an SQL injection for this version of zoneminder, and it is time based.

!\[\[28\_Pebbles\_image003.png]]

&#x20;

!\[\[28\_Pebbles\_image004.png]]

&#x20;

!\[\[28\_Pebbles\_image005.png]]

&#x20;

Testing it with 10 seconds also works out well. From here, it seems that we have to do some form of Blind SQL Injection again.

&#x20;

Ran an SQLMap, although I do plan on doing this onee manually.

Based on the exploit, it seems that the back-end server uses MySQL.

&#x20;

From here, we just need to determine what kinds of tables exist on this server.

&#x20;

SELECT sleep(10) makes it sleep for 10 seconds.

SELECT IF (1=1,sleep(10),'a'\_# causes a true condition to be there.

&#x20;

From rough guessing, it seems that the database would be something along the lines of zm. Because this is a zm database, I expect one table within the zm database to be a user table and somekind of username and password\_hash column.

&#x20;

I figured this would take too long, and instead opted to figure out how to start an RCE from this SQL Injection.

Ideally, we can use SQLMap for an OS shell immediately, but this would not do for OSCP, hence let's do it manually.

&#x20;

First of all, this is a PHP Website, so we need to figure out how to write in some shell code.

!\[\[28\_Pebbles\_image006.png]]

&#x20;

{width="9.572916666666666in" height="1.75in"}

This almost works. However, it seems that there is something to do with the name that is incorrect.

!\[\[28\_Pebbles\_image008.png]]

Alright, we have some form of cmd.php there.

&#x20;

{width="9.4375in" height="1.25in"}

&#x20;

After lots of trying, it seems that I do not know how to get a shell from there. Running an os-shell command on MySQL reveals that the database is operating as a root user.

{width="8.572916666666666in" height="0.90625in"}

&#x20;

So I just let it run and get me the root shell that I wanted. We can download the usual file from here and then execute it accordingly.

Had lots of troubles with getting this shell to work...

{width="9.71875in" height="1.09375in"}

&#x20;

!\[\[28\_Pebbles\_image0012.png]]

&#x20;

!\[\[28\_Pebbles\_image0013.png]]

&#x20;
