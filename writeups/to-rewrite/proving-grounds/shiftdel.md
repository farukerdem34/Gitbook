# Shiftdel

Shiftdel

Wednesday, 30 March 2022

12:55 pm

This is a Linux machine with these services running:

&#x20;

!\[\[41\_Shiftdel \_image001.png]]

&#x20;

!\[\[41\_Shiftdel \_image002.png]]

&#x20;

I see wpscan, hence I should just run wpscan tool to enumerate for me.

Port 8888 is just a phpmyadmin login page.

&#x20;

The wpscan returns loads of vulnerabilities, and the users present are intern and admin:

{width="8.40625in" height="1.75in"}

&#x20;

We don't have any credentials, so I thought it would be wise to let a brute force attempt keep running.

The brute force attempt does not work.

&#x20;

So let's use this exploit.

!\[\[41\_Shiftdel \_image004.png]]

This seems to be the only exploit that does not require any credentials.

&#x20;

We can find out there is one post here that is password protected.

!\[\[41\_Shiftdel \_image005.png]]

&#x20;

This exploit would work this way:

!\[\[41\_Shiftdel \_image006.png]]

From here, we can get the password out.

!\[\[41\_Shiftdel \_image007.png]]

&#x20;

We can login as the intern using these credentials. From there, we can find out more information about the user.

There are other exploits like this one, which allow for RCE on the web server. Since we have credentials, this could be possible to exploit.

!\[\[41\_Shiftdel \_image008.png]]

&#x20;

This exploit was giving me lots of issues, and I could not execute it for some reason. Even MSF was unable to use this exploit, which is a big red flag for me that this is not the case.

&#x20;

Took one hint, and realised that we could delete the .htaccess file that would cause another one to be generated. This would be using the deletion of file exploit found within the wpscan. From there, we are able to view the password for phpmyadmin and find out it is vulnerable to an RCE. This works because of the fact that the htaccess file controls what is being shown on the website and what cannot be shown. We can then read the database through this exploit.

&#x20;

We can reverse shell in as www-data and grab the user flag.

!\[\[41\_Shiftdel \_image009.png]]

&#x20;

For the PE vector, there is this little portion here within the cron jobs:

!\[\[41\_Shiftdel \_image0010.png]]

&#x20;

We can see a custom home being here. From here, we can inject our own binaries which execute and give us a root shell.

&#x20;

!\[\[41\_Shiftdel \_image0011.png]]

&#x20;

&#x20;

&#x20;

&#x20;

&#x20;

&#x20;

&#x20;

&#x20;
