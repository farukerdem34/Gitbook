# Noter

Noter

Sunday, June 19, 2022

7:39 PM

Nmap scan:

!\[\[24\_Noter\_image001.png]]

&#x20;

Port 5000:

!\[\[24\_Noter\_image002.png]]

&#x20;

Creating user:

!\[\[24\_Noter\_image003.png]]

&#x20;

We can play around with this note taking application, and it seems to do nothing of interest.

There's a VIP section that can be accessed.

!\[\[24\_Noter\_image004.png]]

&#x20;

Note Taking:

!\[\[24\_Noter\_image005.png]]

&#x20;

Burpsuite:

!\[\[24\_Noter\_image006.png]]

&#x20;

There's a JWT token being used, and when decoded, there is a username parameter when using the application.

!\[\[24\_Noter\_image007.png]]

&#x20;

Perhaps we can manipulate this. We can change the username to admin and replace the first portion of the token.

Then we can use the browser console to manipulate it.

!\[\[24\_Noter\_image008.png]]

&#x20;

However, this does not work out well.

So cookie manipulation does not really work. What we can do now is attempt other forms of exploits, such as SSTI and XSS.

&#x20;

!\[\[24\_Noter\_image009.png]]

&#x20;

There's this ability to link stuff, so I just tried it with my own IP Address to see if anything is clicking on it.

Doesn't work.

&#x20;

Within the page source, there is this CKEditor thingy.

!\[\[24\_Noter\_image0010.png]]

&#x20;

This version happens to be vulnerable to XSS, but there is no indication that someone else is viewing the stuff we write.

&#x20;

Let's review what we know.

This is an application that uses JWT Tokens, and perhaps there are other users within this machine.

We know this is using CkEditor 4.6.2, which is a dated version that should be vulnerable.

There's also this unknown cookie that we have:

!\[\[24\_Noter\_image0011.png]]

&#x20;

I'm not sure what this cookie is, and it can't be decoded.

&#x20;

Based on these findings, it is probable to assume that we need to modify the JWT token somehow.

Also, the server uses WerkZeug and Python.

&#x20;

!\[\[24\_Noter\_image0012.png]]

&#x20;

Perhaps this is a Flask server.

We can test this with Flask-Unsign and attempt to brute force the session cookie and be able to create one later on.

I was right, as we were able to decode this.

{width="9.822916666666666in" height="1.6145833333333333in"}

&#x20;

So the secret is this, and now we can change the username to other things that wewant.

So now we can create new cookies, but we still need to somehow make it such that we can find another username.

&#x20;

We can actually use Burp to do this.

If we enter a valid user with a wrong password, there is an 'Invalid Login' error that pops up.

!\[\[24\_Noter\_image0014.png]]

&#x20;

If we do so with an invalid user, it would tell us differently.

!\[\[24\_Noter\_image0015.png]]

&#x20;

We can use this to find out users that we want.

&#x20;

!\[\[24\_Noter\_image0016.png]]

&#x20;

!\[\[24\_Noter\_image0017.png]]

&#x20;

I used a wordlist from seclists to do this.

&#x20;

Alternatively, we can set up a bash script to do the same. I tried but was not really able to get a fast response.

&#x20;

Eventually, we would get something like this to pop up.

!\[\[24\_Noter\_image0018.png]]

&#x20;

So Blue is our user here.

{width="9.75in" height="1.0in"}

&#x20;

We can use the browser to execute document.cookie with this new cookie and view the notes to see more stuff.

!\[\[24\_Noter\_image0020.png]]

&#x20;

!\[\[24\_Noter\_image0021.png]]

&#x20;

We can get FTP credentials.

&#x20;

FTP:

{width="7.020833333333333in" height="3.5416666666666665in"}

&#x20;

We can read the policy.pdf to view what this FTP thing is about.

There is a files directory.

{width="13.25in" height="1.9270833333333333in"}

&#x20;

We can view their password policy here, and we know another user is named ftp\_admin.

We can login as FTP\_admin using ftp\_admin@Noter1

&#x20;

{width="8.625in" height="3.5416666666666665in"}

&#x20;

So there's some source code to review now.

VIP Page:

Now that we are this user, we can view the VIP Dashboard.

!\[\[24\_Noter\_image0025.png]]

&#x20;

There are options to upload and import notes.

!\[\[24\_Noter\_image0026.png]]

&#x20;

Comparing files:

{width="5.458333333333333in" height="2.6041666666666665in"}

&#x20;

We can get some credentials here.

&#x20;

Source code review:

When unzipping the files, we can view an app.py.

&#x20;

Import Notes:

{width="12.5in" height="8.760416666666666in"}

&#x20;

From this, we can see how the notes are added directly into the SQL database with no sanitising.

&#x20;

Export Note:

{width="11.916666666666666in" height="8.4375in"}

&#x20;

Interestingly, there seems to be a command that is run through /bin/bash that has no sanitsation. This seems to take the notes through a .pdf and stuff.

&#x20;

Perhaps there is a OS Command Injection here, if we could inject some stuff in there.

Md-to-pdf means that only .md files are allowed to be exported.

!\[\[24\_Noter\_image0030.png]]

&#x20;

It says that it would strip the text and then run a command, meaning we could potentially do command injection here.

So to escape the thing, we would need to first escape the front part and chain commands together with ';.

&#x20;

Afterwards, we would need to cancel out the rest of the command that is to execute within python using # and close the quote we are in.

We can try a reverse shell in python3 since this is using python3.

&#x20;

Shell:

!\[\[24\_Noter\_image0031.png]]

&#x20;

After pressing export, we gain a shell.

{width="5.395833333333333in" height="1.5in"}

&#x20;

Flag:

!\[\[24\_Noter\_image0033.png]]

94739e6fc87526c530de689da7427c1a

&#x20;

PE:

!\[\[24\_Noter\_image0034.png]]

We know from earlier that we had MySQL running as root with those credentials that we found.

!\[\[24\_Noter\_image0035.png]]

&#x20;

We can now do the usual MySQL exploit with raptor\_udf2.c.

!\[\[24\_Noter\_image0036.png]]

&#x20;

When we enter this, we gain a shell as root!

{width="8.229166666666666in" height="0.8125in"}

&#x20;

{width="8.09375in" height="2.15625in"}

&#x20;

Flag:

!\[\[24\_Noter\_image0039.png]]

1259e808a4f616a17e2dbdc24574484f
