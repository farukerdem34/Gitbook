# OpenSource

OpenSource

Friday, June 3, 2022

4:27 PM

Nmap scan:

!\[\[19\_OpenSource\_image001.png]]

&#x20;

&#x20;

Port 80:

!\[\[19\_OpenSource\_image002.png]]

&#x20;

We can download the entire repository here:

!\[\[19\_OpenSource\_image003.png]]

&#x20;

{width="7.875in" height="4.0625in"}

&#x20;

It looks to be some form of Python website.

&#x20;

Source code analysis:

There's a /uploads that supposed to be present.

{width="5.0in" height="0.875in"}

&#x20;

{width="10.916666666666666in" height="6.15625in"}

&#x20;

This would be the file in charge of putting files iwthin the machine.

Trying to access these HTML pages would result in a debug output, and the console is pin protected.

!\[\[19\_OpenSource\_image007.png]]

&#x20;

Views.py:

So in this function, we can firstly see that it uses **get\_file\_name** as the function to retrieve files.

{width="10.604166666666666in" height="2.3854166666666665in"}

&#x20;

/upcloud:

!\[\[19\_OpenSource\_image009.png]]

&#x20;

We can upload files from here.

Creation of test file:

{width="4.28125in" height="1.375in"}

&#x20;

!\[\[19\_OpenSource\_image0011.png]]

&#x20;

From there, we can view the URL of which it has been uploaded to.

Let's try to think, can we overwrite anything regarding this?

They have given us the view.py folder, which perhaps we can upload some kind of shell using this.

&#x20;

Source code reading again:

!\[\[19\_OpenSource\_image0012.png]]

The fatal flaw is this line. We can see that the path of which the variable gets uploaded to is not sanitised. This means that whatever we put the file name as, it would be the path of which it would be uploaded to. This means we can edit something and overwrite it to get what we want.

In this case, because we have access to the source code, we can perhaps append some kind of web shell or something within that and upload it to overwrite the view.py folder.

&#x20;

Since the thing is running in Python, it would seem...fitting that we can upload some kind of python web shell.

!\[\[19\_OpenSource\_image0013.png]]

&#x20;

We can append our own function within this.

This would basically give us a web shell to work with.

&#x20;

!\[\[19\_OpenSource\_image0014.png]]

When uploading this, we can intercept the response and modify the filename parameter, which is vulnerable to directory traversal upload vulnerability (coined this term). Then, we can overwrite the view.py.

&#x20;

Within the source code, we can see that the views.py folder is within the /app/app/ directory.

{width="7.625in" height="0.8541666666666666in"}

&#x20;

The uploads folder is within the /app/public folder.

{width="4.96875in" height="0.7916666666666666in"}

&#x20;

I tried with two different names, one was ..//app//views.py and the other was ..//app//app//views.py, with double back slashing to prevent any cancellation of terms.

&#x20;

Afterwards, I tried to view my upload.

!\[\[19\_OpenSource\_image0017.png]]

&#x20;

Since I was presented this, I assumed that it worked in adding a directory to execute stuff.

We can test this using tcpdump and curl.

{width="7.708333333333333in" height="1.2395833333333333in"}

&#x20;

This would catch any ping requests being made from the server to me.

&#x20;

!\[\[19\_OpenSource\_image0019.png]]

This would basically tell it to ping me once.

&#x20;

{width="9.65625in" height="1.71875in"}

In which case, it worked because we get a ping from the target IP address, which is the 10.129. We have blind RCE.

From here, we can gain a reverse shell easily.

&#x20;

Shell:

!\[\[19\_OpenSource\_image0021.png]]

&#x20;

{width="9.78125in" height="1.96875in"}

&#x20;

We seem to be 'root'. However, this is likely a Flask container of which to run the application.

&#x20;

Upgrading shell:

{width="6.1875in" height="2.2291666666666665in"}

&#x20;

PE:

&#x20;

Cron enumeration:

!\[\[19\_OpenSource\_image0024.png]]

Since we are the root user within this container, we can find the crontab.

There's one that occurs every 15 minutes, which

which is odd.

&#x20;

Checking the ports reveals one interesting port:

!\[\[19\_OpenSource\_image0025.png]]

&#x20;

There's another IP address which is 172.17.0.1.

Going back to .git file for enumeration:

{width="5.25in" height="1.2708333333333333in"}

&#x20;

There was one edit of the dockerfile it seems.

!\[\[19\_OpenSource\_image0027.png]]

&#x20;

There are two branches located within this git repository.

{width="5.708333333333333in" height="3.0416666666666665in"}

&#x20;

We can view more about this branch and its edits.

{width="4.458333333333333in" height="1.5625in"}

&#x20;

When viewing one edit, we actually get credentials.

!\[\[19\_OpenSource\_image0030.png]]

&#x20;

!\[\[19\_OpenSource\_image0031.png]]

&#x20;

It seems that there is a user called dev01 and this is the user.

However, there's no way to SSH in to the user from our machine. Since we have a docker, we probably need to chisel our way in.

!\[\[19\_OpenSource\_image0032.png]]

&#x20;

There was a port 3000 in my nmap scan that was left filtered, hence I tried to chisel to that. Combined with the fact that there was another IP address, this was the command I ran.

&#x20;

!\[\[19\_OpenSource\_image0033.png]]

&#x20;

!\[\[19\_OpenSource\_image0034.png]]

&#x20;

!\[\[19\_OpenSource\_image0035.png]]

&#x20;

There's a sign in.

!\[\[19\_OpenSource\_image0036.png]]

&#x20;

We can sign in using the credentials we found just now.

&#x20;

Within Gitea, there was a home-backup directory.

!\[\[19\_OpenSource\_image0037.png]]

&#x20;

From there, we can gain his SSH key.

!\[\[19\_OpenSource\_image0038.png]]

&#x20;

&#x20;

Shell:

!\[\[19\_OpenSource\_image0039.png]]

&#x20;

We are now the user.

&#x20;

Flag:

!\[\[19\_OpenSource\_image0040.png]]

&#x20;

PE2:

Let's try to view the processes running using pspy64.

!\[\[19\_OpenSource\_image0041.png]]

&#x20;

Within that, there was one interesting process.

!\[\[19\_OpenSource\_image0042.png]]

&#x20;

IT seems that git-remote-http is running this.

Going onto GTFOBins to check on what I can do, it seems that we can use Git Hooks, which are merely shell scripts that would run when the actio nis triggered.

&#x20;

We can create a quick shell.sh

!\[\[19\_OpenSource\_image0043.png]]

&#x20;

!\[\[19\_OpenSource\_image0044.png]]

&#x20;

We can get it here, and then proceed to do these commands.

!\[\[19\_OpenSource\_image0045.png]]

&#x20;

After waiting for a bit, we would gain a shell.

{width="6.520833333333333in" height="1.9375in"}

&#x20;

!\[\[19\_OpenSource\_image0047.png]]

&#x20;

Pwned!

wdaawDAWDAWDAWDAWD
