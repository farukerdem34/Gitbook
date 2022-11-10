# SneakyMailer

SneakyMailer

Wednesday, 23 February 2022

9:50 pm

This is a Linux machine.

The target IP is 10.10.10.197.

My IP is 10.10.16.9.

&#x20;

Nmap scan first.

Preliminary nmap scan shows this.

!\[\[54\_SneakyMailer\_image001.png]]

SMTP! Something interesting :D

&#x20;

!\[\[54\_SneakyMailer\_image002.png]]

I see FTP, so I first tried the anonymous login trick.

&#x20;

Didn't work, moving on!

I wanted to do a banner grab on the SMTP server there, so I did a quick nc connection. Didn't reveal much though.

&#x20;

So let's go to the HTTP server.

Seems to have a different domain name, so I added it into my /etc/hosts.

!\[\[54\_SneakyMailer\_image003.png]]

&#x20;

!\[\[54\_SneakyMailer\_image004.png]]

Going to the web page brought me to some kind of corporate accounts page.

!\[\[54\_SneakyMailer\_image005.png]]

There's this PyPI project, guess I'll remember this for later.

There's a small extension on the right that allows me to view the team and who's in it.

!\[\[54\_SneakyMailer\_image006.png]]

Not sure what this entails, but I did pick out two administrator names, and I suppose we need to keep track of this for the emails on SMTP later.

&#x20;

!\[\[54\_SneakyMailer\_image007.png]]

Not sure how relevant this is either. Viewing the page source, I found this interesting little snippet.

!\[\[54\_SneakyMailer\_image008.png]]

There indeed is a register account portion here.

!\[\[54\_SneakyMailer\_image009.png]]

&#x20;

To keep track, so far we have an index.php, team.php and /pypi/register.php.

&#x20;

!\[\[54\_SneakyMailer\_image0010.png]]

Tried to register for an account. Nothing happened though, my credentials went somewhere else it seems.

Anyways I saved all the emails to a singular file in case I need to brute force something.

!\[\[54\_SneakyMailer\_image0011.png]]

&#x20;

I'm not sure what I'll be using it for.

&#x20;

There's a second HTTP port 8080, and viewing it brought me to the default Nginx page.

!\[\[54\_SneakyMailer\_image0012.png]]

&#x20;

I began to shift my focus to the SMTP port.

!\[\[54\_SneakyMailer\_image0013.png]]

&#x20;

These are the commands we can do, hence we should telnet into it and try to see what we should do perhaps?

{width="4.375in" height="6.21875in"}

I began to enumerate some emails.

Seems that this admin exists on the server.

!\[\[54\_SneakyMailer\_image0015.png]]

In fact, it seems every single email we have listed exists on the network somewhere.

I suppose that we would need to use other methods and not telnet...

&#x20;

So I went onto hacktricks and found the swaks tool which helps to automate a lot of stuff.

!\[\[54\_SneakyMailer\_image0016.png]]

Seems that this tool sends an email to everybody within the machine, and this looks like a phishing email? Cool!

&#x20;

So it seems that if we can change this to our IP address and open up a HTTP port, we can basically check who clicks on our link.

&#x20;

Did exactly that and got this response.

!\[\[54\_SneakyMailer\_image0017.png]]

It seems that somebody is clicking on our links, now I'm wondering if we can potentially get a shell from this, perhaps download something into the web server.

&#x20;

I can see some kind of potential web shell here. Seeing as that it's a POST request and not a GET request, there isn't a way to execute a payload within this.

&#x20;

This seems to be the method into the machine. But first, I wanted to view the response that was being caught by the python server.

&#x20;

Let's open up a NC on port 80.

Resend the command.

The listener port caught this.

!\[\[54\_SneakyMailer\_image0018.png]]

Seems that this includes a password! So paul is the idiot that's clicking my links and stuff.

Decoding it gives us this.

!\[\[54\_SneakyMailer\_image0019.png]]

Looks like nonsense, and this does not work with SSH either.

&#x20;

I tried IMAP to login, seeing HackTricks.

&#x20;

{width="10.09375in" height="1.8020833333333333in"}

This worked! I needed to dump out all of his emails somehow.

&#x20;

Reading the documentation for IMAP.

!\[\[54\_SneakyMailer\_image0021.png]]

&#x20;

This is what I did, and we can see that there's the INBOX.

Let's view this folder.

&#x20;

!\[\[54\_SneakyMailer\_image0022.png]]

&#x20;

{width="8.1875in" height="5.125in"}

Hmm, the first email I retrieved is different.

&#x20;

{width="8.041666666666666in" height="3.0729166666666665in"}

&#x20;

Wonder what this is.

Turns out this is the FTP Server credentials!

!\[\[54\_SneakyMailer\_image0025.png]]

Ok, now we're in here. Let's view the files.

I was looking through and found nothing, until I realised I was an idiot that could just place a reverse shell into this.

Tried as I might, I was unable to trigger the web shell, I looked up a hint and it told me that I needed a dev.sneakycorp.htb and it would work.

!\[\[54\_SneakyMailer\_image0026.png]]

&#x20;

!\[\[54\_SneakyMailer\_image0027.png]]

&#x20;

!\[\[54\_SneakyMailer\_image0028.png]]

Time to look around.

There are two users, both of which I have no access to.

!\[\[54\_SneakyMailer\_image0029.png]]

There was a third website that was present.

!\[\[54\_SneakyMailer\_image0030.png]]

I could not access it however, so this left me with very little options on how to proceed.

I knew that there were two ports, one on 80 and one on 8080.

&#x20;

However, I could not access this website on both of them.'

I opened Burpsuite and wanted to test it out.

Surprisingly, this worked.

(I later learnt that this website is only accessible using a proxy on port 8080! Which my Burp runs on! Accidental LOL)

&#x20;

Anyways, this brings us to here.

!\[\[54\_SneakyMailer\_image0031.png]]

Checked around for some vulnerabilities, of which there were a few.

But first, I ran the command given to me, after all, who wouldn't?

However, those links to the packages required some kind of authentication.

&#x20;

I checked out the ports that were open, and found that there was a port 5000 for some reason.

!\[\[54\_SneakyMailer\_image0032.png]]

Port tunneling is something we are gonna have to do as well.

&#x20;

I looked at the directory of the thing first, and also found this hash.

!\[\[54\_SneakyMailer\_image0033.png]]

Looks to be the credentials I need for the authorisation.

&#x20;

Let's crack it.

{width="9.854166666666666in" height="2.0729166666666665in"}

So this is something new. Let's see if it works on the pypi server.

Well it works, but clearly not something that was intended.

&#x20;

!\[\[54\_SneakyMailer\_image0035.png]]

Let's try su into low.

Does not work. It seems that we need to create our own packages in order to use this server properly.

&#x20;

Well, this would involve Googling around, and this made me end up [here](https://github.com/Ayrx/malicious-python-package).

This is basically a malicious python package that uses the setup.py to execute arbitrary code.

&#x20;

So, after more googling, I found out how to create a simple package [here](https://github.com/Ayrx/malicious-python-package).

&#x20;

After all the steps, I ended with something like this.

!\[\[54\_SneakyMailer\_image0036.png]]

More research told me that it was possible for us to create malware and stuff using this trick, through tricking people into installing our packages, can't trust anybody these days.

&#x20;

This would generate a quick python package that we can upload to the server. I followed the instructions exactly and appended this code into setup.py.

!\[\[54\_SneakyMailer\_image0037.png]]

This was my SSH key, and this let me SSH into the machine.

&#x20;

Once that was done, I had to find a way to upload this to the server.

This is when I looked at some Python documentation and found this.

{width="3.6979166666666665in" height="2.5729166666666665in"}

These are the contents for the \~/.pypirc file that has to be created in order to upload our package. This would allow us to key in the URL and that password we cracked earlier with john.

&#x20;

This has to be created in said folder, so use touch to make it.

&#x20;

Afterwards, once this is all done, we need to find a way to upload it.

!\[\[54\_SneakyMailer\_image0039.png]]

This method on the python documentation uploaded the file and from there, it didn't work anymore.

&#x20;

Why?

I have no idea.

Looking up a walkthrough, this determined that I had done stuff correctly but it seems to use sdist instead of twine.

Changed the entire process. It seems that the server also uses .tar.gz files, which is compiled using the same method but just using sdist.

&#x20;

I'm not entirely sure why that does not work.

&#x20;

Anyways, running the same stuff, we will be able to SSH in.

!\[\[54\_SneakyMailer\_image0040.png]]

&#x20;

I did a quick sudo -l, and found out I could run pip3.

!\[\[54\_SneakyMailer\_image0041.png]]

This was the command found on GTFObins for PIP.

&#x20;

So I Ran the same thing with Pip3 instead and this gave me root.

!\[\[54\_SneakyMailer\_image0042.png]]

Pwned.

&#x20;

To me, a hard box because of my lack of familiarity with Python packages and general programming.

&#x20;

Plus there were a lot of steps required, but overall pretty cool when dealing with phishing emails, then SMTP commands, then FTP, then Python and finally sudo misconfigurations.

&#x20;

&#x20;
