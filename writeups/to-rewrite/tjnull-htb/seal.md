# Seal

Seal

Tuesday, 1 March 2022

12:06 pm

This is a Linux machine.

The target IP is 10.10.10.250.

My IP is 10.10.16.9.

&#x20;

Enumerate the machine!

A quick nmap scan reveals these ports.

!\[\[62\_Seal\_image001.png]]

HTTPS!

When viewing the website, this is what we are greeted with.

!\[\[62\_Seal\_image002.png]]

Interesting. Let's take a look at the certificate.

&#x20;

This gives us an email address that we perhaps have to use later.

!\[\[62\_Seal\_image003.png]]

Anyways, when we search stuff, it appears up here.

!\[\[62\_Seal\_image004.png]]

Perhaps this could be an outlet for injections.

&#x20;

Ran a quick directory enumeration to test it out.

!\[\[62\_Seal\_image005.png]]

&#x20;

So there's an admin and manager directory there, which should be interesting to look at.

&#x20;

While keying in the admin, I spelt it wrongly and this told me that this is running on tomcat.

!\[\[62\_Seal\_image006.png]]

Cool. WAR file again!

Interestingly, when trying to access the admin portal on the HTTPS server, I get a 403.

&#x20;

So we have seen this website, let's look at port 8080.

This redirects me to a Gitbucket sign in page.

!\[\[62\_Seal\_image007.png]]

There's a create account, which I would gladly do.

Logged in, and this is what I get.

!\[\[62\_Seal\_image008.png]]

Anyways, looking at the pull requests, we can see the git repository of the entire website.

!\[\[62\_Seal\_image009.png]]

From here, we can go to the tomcat-users.xml and check the history, to find this.

&#x20;

!\[\[62\_Seal\_image0010.png]]

Alright, we have some credentials.

SSH does not work with this.

Looking at some other codes, we can see that there is the nginx folder there to, and I went to view the sites-available.

&#x20;

!\[\[62\_Seal\_image0011.png]]

This little snippet of code explains why I was served a 403.

&#x20;

Looking around more, found this as well.

!\[\[62\_Seal\_image0012.png]]

Mutual Authentication? Hmm.

&#x20;

Okay, so this is all that I have gotten from this website.

Right now, I wanted to find some kind of misconfiguration. Clearly we need to access that web manager but I can't do so normally. I began my Googling on Tomcat vulnerabilities and possible directory traversal outlets.

&#x20;

I found something like this.

!\[\[62\_Seal\_image0013.png]]

So there is a vulnerability somehow...and I need to find a way to access it.

!\[\[62\_Seal\_image0014.png]]

So this works somehow. Tomcat threats the sequence of ..; or something.

&#x20;

I tried with ; and this worked!

!\[\[62\_Seal\_image0015.png]]

&#x20;

!\[\[62\_Seal\_image0016.png]]

Now I can log in easily.

&#x20;

And we're in!

!\[\[62\_Seal\_image0017.png]]

&#x20;

From here, generate a WAR file using msfvenom.

&#x20;

!\[\[62\_Seal\_image0018.png]]

&#x20;

!\[\[62\_Seal\_image0019.png]]

Deploy it.

&#x20;

And then get walled.

!\[\[62\_Seal\_image0020.png]]

Weird...

I don't know why it's blocking me.

&#x20;

So I tried the exploit I had already found. I just appended it everywhere, and also changed the page source so that it would follow suit.

&#x20;

Eventually I changed it to .; to follow what I saw in the github.

&#x20;

!\[\[62\_Seal\_image0021.png]]

I got it to upload.

From there, execute it and we have a shell.

&#x20;

!\[\[62\_Seal\_image0022.png]]

Stabilise it and we are in.

!\[\[62\_Seal\_image0023.png]]

Luis is the only user here.

&#x20;

!\[\[62\_Seal\_image0024.png]]

&#x20;

Within the /var/www folder I found some keys, looks to be the certificates used for the website.

&#x20;

!\[\[62\_Seal\_image0025.png]]

Within the /opt directory, I found this backups folder.

&#x20;

!\[\[62\_Seal\_image0026.png]]

Let's take a look shall we.

So within playbook, there seems to be some kind of script running, basically taking the admin dashboard and putting it in the archive.

!\[\[62\_Seal\_image0027.png]]

And this seems to be repeating every minute or so.

&#x20;

!\[\[62\_Seal\_image0028.png]]

This cronjob is run by luis, and I don't think we are able to write into it.

&#x20;

When going into the dashboard directory, we can find one file that is indeed writable.

!\[\[62\_Seal\_image0029.png]]

So we can put whatever we want into this and perhaps get something out.

&#x20;

I wanted to take a look at the .gz files being generated, so I took one to /tmp to experiment on it.

&#x20;

I took it there, and tried to unzip it basically.

Inside there were the folders as per normal.

&#x20;

Now I was thinking. Could I perhaps change the directory within the yml file so that it would keep a completely different file?

Then I remembered that symbolic links are used to link files together basically, so I did this to get the home directory of luis into that of the uploads folder, the one folder I can write.

!\[\[62\_Seal\_image0030.png]]

&#x20;

This should get the home directory of luis within the exploit there, and I'll be on my way.

&#x20;

Waited for the next round of archiving, and took the latest file and unzipped it as such:

!\[\[62\_Seal\_image0031.png]]

&#x20;

Yes I know there were a lot of errors.

&#x20;

From there, there should be the home directory of luis within the uploads folder.

However, this was not working for me...

Looked up a guide and identified that I was doing the right thing.

I tried again, and this time it worked.

!\[\[62\_Seal\_image0032.png]]

Within this, there was also the SSH keys and stuff.

&#x20;

I copied over the id\_rsa private key.

!\[\[62\_Seal\_image0033.png]]

&#x20;

Transfer it over and SSH using it.

!\[\[62\_Seal\_image0034.png]]

I ran a sudo -l, and this is what Luis can do.

&#x20;

!\[\[62\_Seal\_image0035.png]]

&#x20;

I remember seeing the run.yml there, and I wondered if I could change that.

I grabbed it to create my own to test.

!\[\[62\_Seal\_image0036.png]]

&#x20;

This worked.

{width="9.104166666666666in" height="3.2916666666666665in"}

Now all I need to do is inject some reverse shell in there and I'm good.

&#x20;

!\[\[62\_Seal\_image0038.png]]

Here we go.

{width="11.479166666666666in" height="1.25in"}

&#x20;

&#x20;

!\[\[62\_Seal\_image0040.png]]

Pwned.
