# Cassios

Cassios

Sunday, 3 April 2022

5:03 pm

This is a Linux machine with these services:

!\[\[51\_Cassios\_image001.png]]

&#x20;

Port 80 has nothing, while port 8080 has a login page.

&#x20;

Port 80:

!\[\[51\_Cassios\_image002.png]]

&#x20;

Gobuster scan of port 80:

!\[\[51\_Cassios\_image003.png]]

&#x20;

!\[\[51\_Cassios\_image004.png]]

&#x20;

In the backup\_migrate file, there is this.

!\[\[51\_Cassios\_image005.png]]

&#x20;

Within the /download file, there's a zip file located here.

{width="8.447916666666666in" height="1.5in"}

&#x20;

If we look through the files, we can just see this is basically the web assets.

&#x20;

/recycler:

We can download this .tar file and begin to analyse it.

!\[\[51\_Cassios\_image007.png]]

&#x20;

{width="9.635416666666666in" height="2.7916666666666665in"}

&#x20;

This seems to be the some other web page that is being used, and I'm particular interested in the Java Based Configurations that are here.

!\[\[51\_Cassios\_image009.png]]

&#x20;

We can extract these:

{width="5.0625in" height="0.6666666666666666in"}

&#x20;

&#x20;

Within the WebSecurtiyConfig.java, we can find some credentials.

{width="5.5625in" height="0.6041666666666666in"}

&#x20;

!\[\[51\_Cassios\_image0012.png]]

Cool.

&#x20;

Port 8080:

!\[\[51\_Cassios\_image0013.png]]

&#x20;

We can sign in:

!\[\[51\_Cassios\_image0014.png]]

&#x20;

!\[\[51\_Cassios\_image0015.png]]

&#x20;

Source Code analysis:

{width="7.0625in" height="4.083333333333333in"}

&#x20;

We can see that it takes some backups from the user's home directory.

&#x20;

When looking further into this file, we can see that the values within the file are being used within the host here.

!\[\[51\_Cassios\_image0017.png]]

&#x20;

So basically, there is input from the recycler.ser that is being put into the dashboard, without being sanitised. The input of the file is being unserialised without user input validation, meaning we could theoretically just put our own file that executes arbitrary commands and then reload the dashboard to cause it to input stuff into the web page.

&#x20;

Enum4linux scan:

{width="3.8854166666666665in" height="0.5416666666666666in"}

&#x20;

{width="9.791666666666666in" height="2.0729166666666665in"}

Null sessions are allowed, and I expect some form of shares being available for us to look into.

&#x20;

!\[\[51\_Cassios\_image0020.png]]

&#x20;

Shares:

We can login.

{width="8.21875in" height="2.9791666666666665in"}

&#x20;

We can see how the recycler.ser file is empty.

Interestingly, we are allowed to put files as well.

{width="4.177083333333333in" height="0.6666666666666666in"}

&#x20;

!\[\[51\_Cassios\_image0023.png]]

&#x20;

!\[\[51\_Cassios\_image0024.png]]

&#x20;

This means we can overwrite the recycler.ser file with our own that could generate some malicious Java code.

This basically confirms that we need some deserialisation to exploit this machine.

&#x20;

Downloading of all files:

!\[\[51\_Cassios\_image0025.png]]

&#x20;

From the files, we can see the commons-collections used.

!\[\[51\_Cassios\_image0026.png]]

&#x20;

Ysoserial:

{width="6.6875in" height="0.7916666666666666in"}

&#x20;

!\[\[51\_Cassios\_image0028.png]]

&#x20;

Then we can transfer this file over to the machine.

!\[\[51\_Cassios\_image0029.png]]

&#x20;

Shell:

!\[\[51\_Cassios\_image0030.png]]

Click check status to execute the exploit.

{width="7.0625in" height="1.9583333333333333in"}

&#x20;

Flag:

!\[\[51\_Cassios\_image0032.png]]

&#x20;

PE:

!\[\[51\_Cassios\_image0033.png]]

&#x20;

!\[\[51\_Cassios\_image0034.png]]

&#x20;

!\[\[51\_Cassios\_image0035.png]]

&#x20;

There's a clear exploit path here.

!\[\[51\_Cassios\_image0036.png]]

&#x20;

We are allowed to sudoedit files from there, and this sudo conf has 2 wildcards that are very dangerous to have. This is vulnerable to the symlink exploit, whereby we can create some kind of symlink with this file here and another file like /etc/passwd and edit it to add our own user.

&#x20;

To do this, we would need to upgrade our shell using stty.

{width="4.28125in" height="2.0104166666666665in"}

&#x20;

We would be able to gain the privileges of being use our arrow keys or something.

&#x20;

Creation of fake user:

{width="5.5in" height="0.78125in"}

&#x20;

{width="7.5in" height="0.8229166666666666in"}

&#x20;

{width="6.270833333333333in" height="0.7604166666666666in"}

&#x20;

!\[\[51\_Cassios\_image0041.png]]

&#x20;

!\[\[51\_Cassios\_image0042.png]]

&#x20;

We can now edit the /etc/passwd file to add our own user within the machine.

Press 'I' to go into insert mode and add our user at the bottom.

Then hit ESC and :wq to exit and save our changes.

!\[\[51\_Cassios\_image0043.png]]

&#x20;

Then just su.

!\[\[51\_Cassios\_image0044.png]]

&#x20;

Flag:

!\[\[51\_Cassios\_image0045.png]]

&#x20;

Pwned!

Pretty cool box.

&#x20;

&#x20;

&#x20;
