# Apex

Apex

Thursday, 31 March 2022

12:08 am

Linux machine with these ports:

!\[\[46\_Apex \_image001.png]]

&#x20;

Interesting, there's port 445 here.

{width="9.75in" height="2.5104166666666665in"}

And we can read some of it.

&#x20;

When logged in, it seems that it tells us that the web portal is running on OpenEMR.

!\[\[46\_Apex \_image003.png]]

&#x20;

There are lot of vulnerabilities regarding this:

!\[\[46\_Apex \_image004.png]]

&#x20;

I can see that the website on port 80 is one that includes a lot of medical related stuff.

!\[\[46\_Apex \_image005.png]]

&#x20;

Enumerating more on port 3306, we can find out more information about the MySQL server which is running

&#x20;

!\[\[46\_Apex \_image006.png]]

There seem to be valid null credentials and stuff to this server, and we can take note of that salt as well.

&#x20;

After some directory enumeration, I found this filemanager directory here.

!\[\[46\_Apex \_image007.png]]

&#x20;

We are not allowed to upload any php files though.

!\[\[46\_Apex \_image008.png]]

Interestingly, we can trigger a RFI using the from URL portion.

!\[\[46\_Apex \_image009.png]]

However, it seems to block most forms of uploads.

&#x20;

When we do successfully upload something, it appears here:

!\[\[46\_Apex \_image0010.png]]

&#x20;

!\[\[46\_Apex \_image0011.png]]

You can see what I did here.

&#x20;

Now that we know that this has a rather weak firewall, we can try to upload it properly using the nullbyte trick. However, this does not lead to much.

&#x20;

This version was running Responsive File Manager, which had a directory traversal exploit for it.

&#x20;

So as it turns out, after resorting to some hints, we actually are meant to use smb share to read files. How the directory traversal exploit works is that it copies and pastes the file we want into the directory. This would let us read the config files that the .htpasswd files block us from. From here, we can read some SQL Credentials located within the OpenEMR files, of which the latter's repository can be found online.

&#x20;

After reading these files, we are able to log into an MariaDB server and get out some passwords. After cracking the hash for the admin user of the OpenEMR server, we can easily log in and exploit the RCE using these credentials to gain a reverse shell.

&#x20;

After which, the password that we cracked earlier was reused for the root user and we can pwn the box that way.
