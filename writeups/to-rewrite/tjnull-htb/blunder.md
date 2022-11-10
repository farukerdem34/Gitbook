# Blunder

Blunder

Tuesday, 15 February 2022

3:37 pm

This is a Linux machine.

My IP is 10.10.16.9.

The target IP is 10.10.10.191.

&#x20;

Quick nmap scan reveals that there are two ports, HTTP and FTP are detected. on this machine.

I did a full port scan as well, just to check.

!\[\[39\_Blunder\_image001.png]]

&#x20;

Visiting the web page reveals to me that this is just a bunch of text about facts of life.

!\[\[39\_Blunder\_image002.png]]

At the bottom of the page however, I found one interesting line.

&#x20;

!\[\[39\_Blunder\_image003.png]]

So we sort of know the engine, and looking at the copyright year, it's safe to assume that this is an older engine that is being used. This ends up being nothing of use.

&#x20;

Ran a quick directory enumeration and found lots of directories. For now, I'll investigate these directories while loading a bigger scan using the medium directory list available.

&#x20;

!\[\[39\_Blunder\_image004.png]]

I ran a larger directory scan and it revealed much more about the target.

Here's the admin panel protected by a log in.

!\[\[39\_Blunder\_image005.png]]

{width="5.34375in" height="1.59375in"}

&#x20;

The install.php and todo.txt is very interesting. Taking a look at it reveals this.

&#x20;

!\[\[39\_Blunder\_image007.png]]

&#x20;

!\[\[39\_Blunder\_image008.png]]

Ok, so there's Bludit on the machine, and some user called fergus that can upload pictures to the website.

&#x20;

Looking up Bludit, it seems that it is a web application that can be used to build the tools fast. It's open source, so this means that we should be able to look up some open files such as passwords or something.

&#x20;

I was taking a look at the page source to try to find some information, as well as sorting out what I already had.

There's this little gem here.

!\[\[39\_Blunder\_image009.png]]

&#x20;

So this should be running Bludit 3.9.2. Additionally, I also noted that there's some form of .git files present on the website.

This version parameter is present on the JS versions as well.

!\[\[39\_Blunder\_image0010.png]]

&#x20;

{width="6.375in" height="2.1979166666666665in"}

&#x20;

Looks like there are some forms of bypassing the authentication, and I'm going to take the first exploit there. I already know that the user is called fergus, and there's some form of brute forcing going on. I used the first one, which was a Python script.

&#x20;

Next, I was thinking of just using rockyou, but seeing as to how slow this password cracking was, I knew that the password was not in here. Normally HTB boxes take a short while to brute force if they require you to.

&#x20;

I googled how to create our own password lists, and came across CeWl, which is a Linux tool.

!\[\[39\_Blunder\_image0012.png]]

&#x20;

So this creates passwords for the websites. Pretty cool.

Used this to create my password list based on the manual.

&#x20;

!\[\[39\_Blunder\_image0013.png]]

&#x20;

Used this password list and cracked the password.

&#x20;

!\[\[39\_Blunder\_image0014.png]]

&#x20;

!\[\[39\_Blunder\_image0015.png]]

Interestingly, rockyou.txt does not even have this password within it.

&#x20;

!\[\[39\_Blunder\_image0016.png]]

Log into the admin panel.

{width="9.822916666666666in" height="7.166666666666667in"}

So we know that fergus is in charge of putting the pictures on the website. Hence, we would need to find a way to upload some form of image somewhere with a reverse shell to our machine.

&#x20;

Googled around for RCEs related to Bludit 3.9.2.

&#x20;

Found [this](https://github.com/nthR00t/CVE-2019-16113) script on Github.

Ran it, it's a pretty good script to use for this.

Anyways, I ran it and it gave me a reverse shell.

&#x20;

!\[\[39\_Blunder\_image0018.png]]

&#x20;

!\[\[39\_Blunder\_image0019.png]]

&#x20;

Taking a look at the code, we can see that there was an image upload vulnerability. And all it does is just run the script located within the uploaded image.

&#x20;

!\[\[39\_Blunder\_image0020.png]]

Stabilise the shell, and proceed to look around.

&#x20;

Looked around at the bl-content and found a databases file, and inside it there's a user.php file.

!\[\[39\_Blunder\_image0021.png]]

&#x20;

Seems that there's some form of password and admin there. There are two users, called shaun and hugo within the machine, and hugo is the one with the user.txt file.

!\[\[39\_Blunder\_image0022.png]]

&#x20;

The password looks like some form of hash that has been salted.

Anyways, I knew there was another file within /var/www, called bludit-3.10.0a, and this also had another database file.

I took a look at the file and saw that there was another password and account for hugo.

!\[\[39\_Blunder\_image0023.png]]

Cool!

&#x20;

Crack the hash accordingly.

!\[\[39\_Blunder\_image0024.png]]

Now we should be able to access hugo.

&#x20;

!\[\[39\_Blunder\_image0025.png]]

Get the user flag, and let's continue with our privilege escalation.

&#x20;

Now we can take a look at what we can sudo.

&#x20;

!\[\[39\_Blunder\_image0026.png]]

&#x20;

So turns out this means that we are unable to run /bin/bash as root, hence the !root.

However, it does mean that we are able to run the sudo /bin/bash command, and sudo also does not check for the specified user id and executes arbitrary user id with the sudo privilege.

&#x20;

Hence we can easily bypass the !root, as sudo does not check for that. Hence, we can execute /bin/bash as root and privilege escalate.

&#x20;

!\[\[39\_Blunder\_image0027.png]]
