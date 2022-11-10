# Spaghetti

Spaghetti

Sunday, 3 April 2022

11:43 am

Nmap scan:

!\[\[49\_Spaghetti\_image001.png]]

&#x20;

&#x20;

Port 80:

!\[\[49\_Spaghetti\_image002.png]]

&#x20;

Spaghetti mail.beta, take note of that domain name.

&#x20;

!\[\[49\_Spaghetti\_image003.png]]

There's another one down here.

&#x20;

Port 8080:

!\[\[49\_Spaghetti\_image004.png]]

&#x20;

When we try to login using default credentials, there is a CSRF check that is failed.

!\[\[49\_Spaghetti\_image005.png]]

&#x20;

Interesting, because there are exploits to guess this token.

!\[\[49\_Spaghetti\_image006.png]]

&#x20;

Interesting, however this is rather uneventful.

{width="7.0in" height="2.6875in"}

&#x20;

Irc might have the details that we need.

!\[\[49\_Spaghetti\_image008.png]]

&#x20;

There are 2 users in this table.

!\[\[49\_Spaghetti\_image009.png]]

&#x20;

Guess the admin is this, bt not notable.

!\[\[49\_Spaghetti\_image0010.png]]

&#x20;

This has been disabled, so nothing there to look at.

{width="6.583333333333333in" height="1.03125in"}

&#x20;

There are channels to go to, and there are some usernames.

!\[\[49\_Spaghetti\_image0012.png]]

&#x20;

We can see some weird string there, which does not look very legit.

&#x20;

Anyways, we can definitely sign in to the thing using a GUI called pidgin and be able to actually chat with the channel that we are in.

&#x20;

!\[\[49\_Spaghetti\_image0013.png]]

&#x20;

!\[\[49\_Spaghetti\_image0014.png]]

&#x20;

So we can basically send emails, perhaps we can find out more about this administrator.

Since we can send emails, I might as well try some phishing emails.

&#x20;

What puzzles me is that hidden stuff that is sent to me as well, the nunch of stars is not there within the default script of the command.

&#x20;

Looking at the wireshark input, we can at least find one user, which is the spaghetti user.

!\[\[49\_Spaghetti\_image0015.png]]

&#x20;

I suppose he's the one that is running this IRC bot service.

Through the github page that is sent, we can find the user's Github account.

&#x20;

!\[\[49\_Spaghetti\_image0016.png]]

&#x20;

Perhaps from here, we can download the repository and then view all of the different logs.

!\[\[49\_Spaghetti\_image0017.png]]

&#x20;

Which don't reveal much at all.

&#x20;

The source code reveals quite a lot.

!\[\[49\_Spaghetti\_image0018.png]]

&#x20;

So there's a command being issued and there is regex being used to make sure there are no malicious characters being input into the server.

The regex should be the key to this, and we can see it doesn't block out a whole load of characters, meaning we can possibly escape and have some RCE done on the machine through sending a crafted email.

&#x20;

The regex seems to only check the email address itself, and not the actual message. The script uses mail -s and puts the arguments in a quote before executing the command.

&#x20;

WIP, but we at least have found the version and what to do. We can already see a command injection point there.

&#x20;

Let's attempt to send one email first, and see what is the response.

&#x20;

!\[\[49\_Spaghetti\_image0019.png]]

&#x20;

So this isn't a valid email, and hence it returns false.

We need to figure out what is that regex first and find a valid email to fit it.

&#x20;

We can take the code out, and begin to analyse and test potential emails with it here.

{width="6.375in" height="3.9791666666666665in"}

&#x20;

So we have a valid email here.

{width="4.8125in" height="0.8854166666666666in"}

&#x20;

But it still doesn't work...

!\[\[49\_Spaghetti\_image0022.png]]

&#x20;

For some reason, the email keeps messing up which is very annoying.

Then I realised im an idiot.

!\[\[49\_Spaghetti\_image0023.png]]

&#x20;

So we have successfully sent an email, let's try some command injection.

!\[\[49\_Spaghetti\_image0024.png]]

&#x20;

The body portion, which is the description, can be used for command injection.

&#x20;

Sending this seems to crash the server,.

!\[\[49\_Spaghetti\_image0025.png]]

&#x20;

And I didn't get my reverse shell.

So we perhaps we can experiment with some quotation escapes.

&#x20;

!\[\[49\_Spaghetti\_image0026.png]]

&#x20;

{width="8.3125in" height="1.84375in"}

&#x20;

Flag:

!\[\[49\_Spaghetti\_image0028.png]]

&#x20;

We can find the password for the bot as well.

!\[\[49\_Spaghetti\_image0029.png]]

&#x20;

PE:

We can run LinPEAS on this thing and determine potential vectors of which we can use.

&#x20;

Interesting output:

!\[\[49\_Spaghetti\_image0030.png]]

&#x20;

Mysql is on the machine.

There are cronjobs, though they may be ran by the user itself.

!\[\[49\_Spaghetti\_image0031.png]]

&#x20;

Within the /opt directory, there's this script and it is owned by root.

!\[\[49\_Spaghetti\_image0032.png]]

&#x20;

!\[\[49\_Spaghetti\_image0033.png]]

&#x20;

Seems that this runs every so often, and it seems to use the mysql conf file within root, which we cannot read.

&#x20;

Then it does some SQL queries, and does some stuff.

This is indeed running based on pspy64.

&#x20;

!\[\[49\_Spaghetti\_image0034.png]]

&#x20;

Running every minute or so.

Then remember that we have postfixadmin there, and we could potentially use this to do SQL Injection to write a shell.

!\[\[49\_Spaghetti\_image0035.png]]

&#x20;

We can find the database password here.

{width="6.1875in" height="2.6666666666666665in"}

&#x20;

With these credentials, we can login.

{width="7.875in" height="3.8854166666666665in"}

&#x20;

We can find the password of the admin user and try to crack the hash.

!\[\[49\_Spaghetti\_image0038.png]]

&#x20;

Understanding that there is a cronjob executed by root, we would need to figure out how to inject some commands through the username parameter that is passed into the script.

&#x20;

We can first generate a quick script that would be used for a shell.

!\[\[49\_Spaghetti\_image0039.png]]

&#x20;

Then download it to the seever and make it executable.

!\[\[49\_Spaghetti\_image0040.png]]

&#x20;

Then we just have to update the username and the date to something that would trigger the cronjob to execute the script.

Afterwards, we just wait around for the script to execute itself, and we would be able to gain a root shell on our listener port.

!\[\[49\_Spaghetti\_image0041.png]]

&#x20;

!\[\[49\_Spaghetti\_image0042.png]]

&#x20;

!\[\[49\_Spaghetti\_image0043.png]]

&#x20;

!\[\[49\_Spaghetti\_image0044.png]]

&#x20;

Flag:

!\[\[49\_Spaghetti\_image0045.png]]

&#x20;

&#x20;
