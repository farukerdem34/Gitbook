# Sybaris

Sybaris

Friday, 18 March 2022

10:58 am

This machine basically had some stuff regarding redis.

&#x20;

There was an anonymous login using FTP, and from there we can upload a module for redis that allows for RCE.

&#x20;

From there, we can get back a reverse shell as user pablo.

!\[\[19\_Sybaris\_image001.png]]

&#x20;

Check the LDD of the cronjob and determine that it's missing one utils.so.

&#x20;

From there use export LD\_LIBRARY\_PATH=/usr/local/lib/dev and place a utils.so that grants a reverse shell back to us.

&#x20;

Open a listener port and just wait.

Take note that for me I used port 80, as the rest of the ports were not allowed to have connections from the machine.

&#x20;

Was taking too long for the shell to execute, so I skipped first.

Did everything as per normal injection of shared objects.

&#x20;
