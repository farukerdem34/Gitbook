# Networked

Networked

Monday, 7 February 2022

10:27 pm

This is a Linux machine.

The target IP is 10.10.10.146.

My IP is 10.10.16.5

&#x20;

Preliminary scan reveals there is SSH, HTTP and HTTPS open on the web server.

Detailed scan reveals this.

!\[\[21\_Networked\_image001.png]]

&#x20;

HTTPS port is closed it seems.

Directory enumeration reveals this.

!\[\[21\_Networked\_image002.png]]

&#x20;

!\[\[21\_Networked\_image003.png]]

&#x20;

!\[\[21\_Networked\_image004.png]]

&#x20;

Going to the web page reveals this.

!\[\[21\_Networked\_image005.png]]

&#x20;

!\[\[21\_Networked\_image006.png]]

&#x20;

Interesting comment, seems to point to /uploads. This directory has nothing. The most interesting would be /backup and /upload.php.

&#x20;

/backup contains a .tar file that has all the source code for each php present.

!\[\[21\_Networked\_image007.png]]

&#x20;

Upload.php allows for us to upload something.

!\[\[21\_Networked\_image008.png]]

&#x20;

From here, I suppose we're supposed to look at the code and determine how to get stuff out of it. Let's take a look at upload.php.

&#x20;

The directory would be /var/www/html/lib.php, which as of now contains an empty thing, presumably because we have not uploaded anything. Trying to upload anything results in an error. Looking at the code, it seems to check the file size, which has to be above 60,000 and also the file type, which has to follow a certain format.

{width="10.572916666666666in" height="1.75in"}

&#x20;

{width="6.0625in" height="0.8229166666666666in"}

&#x20;

These are the valid extensions that can be there. When we do a check on the code, we can see that it does a substr\_compare, which would check for the name of the file having the correct extension. However, we can see that the code only checks for whether the extension is present, and it does not check whether it is the true extension.

&#x20;

This would allow us to perhaps make a file with double extensions. From here we can upload a web shell!

{width="7.802083333333333in" height="1.7708333333333333in"}

&#x20;

Sick, so we just need to include something .php.png for example.

\* \*

!\[\[21\_Networked\_image0012.png]]

&#x20;

Created a quick script, and let's try uploading it. Make sure that it is of a certain size! I tested it before doing all of this on this black png, and it seems that it would be uploaded onto photos.php.

!\[\[21\_Networked\_image0013.png]]

I uploaded the web shell next.

&#x20;

When I try to view it, I got a shell on a listener port!

!\[\[21\_Networked\_image0014.png]]

&#x20;

!\[\[21\_Networked\_image0015.png]]

(for some reason it wouldn't let me spawn in a nice shell :( )

&#x20;

Afterwards let's begin exploring. There's one user called **guly**. Within his files, there is a check\_attack.php and a crontab.guly.

!\[\[21\_Networked\_image0016.png]]

Checking out the .php file, it seems that there is a log path and stuff within it to /tmp/attack.log.

{width="6.46875in" height="7.90625in"}

&#x20;

Not too sure what this does.

Looking at the crontab file, it seems to be executing the php file in a set interval repeatedly.

&#x20;

!\[\[21\_Networked\_image0018.png]]

&#x20;

This PHP file seems to be checking on whether there is malicious stuff basically, I'm not very familiar with php so idk...

What I do know is that this has netcat installed, and I can make use of this file to give me a reverse shell back to my machine!

&#x20;

The path stated in the file seems to be the uploads file, so let's just go there. It's clear we need to create an executable within the /uploads folder.

&#x20;

Since we need to write something to the folder, we cannot use /bin/bash. We need to use touch.

&#x20;

!\[\[21\_Networked\_image0019.png]]

&#x20;

This would give us a shell eventually. The -c command would check whether the file is present, and it does not create files.

!\[\[21\_Networked\_image0020.png]]

&#x20;

Grab the flag. Then spawn a nicer TTY shell.

&#x20;

I ran a sudo -l, as I always do and see that the user guly can run the following files.

!\[\[21\_Networked\_image0021.png]]

&#x20;

{width="7.0in" height="4.708333333333333in"}

&#x20;

Idk what this does, let's run it.

&#x20;

{width="12.541666666666666in" height="3.3854166666666665in"}

&#x20;

Intriguing, it checks for device guly0. It writes to a specific file, so let's see what gets outputted.

!\[\[21\_Networked\_image0024.png]]

I tried injecting commands and actually got output.

!\[\[21\_Networked\_image0025.png]]

&#x20;

This seems to be able to run stuff, if it returned the proper things, so let's try injecting /bin/bash. And just like that, pwned.

{width="4.895833333333333in" height="3.40625in"}

&#x20;

The touch command is great! Great for writing things for reverse shells.

&#x20;

Looking again at the code, there was actually a regex prevention, but it does not block a lot of stuff. This led to the exploit happening.

&#x20;

&#x20;
