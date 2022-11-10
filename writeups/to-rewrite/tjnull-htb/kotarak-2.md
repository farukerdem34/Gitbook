# Kotarak

**Kotarak**

Sunday, 27 February 2022

11:30 am

This is an **Insane** Linux box.

My IP is 10.10.16.9.

The target IP is 10.10.10.55.

&#x20;

Let's enumerate out this box on all ports first.

!\[\[60\_Kotarak\_image001.png]]

Okay, now let's scan these ports.

&#x20;

!\[\[60\_Kotarak\_image002.png]]

I already see that we can do a PUT command on these ports, which is risky because that could be the method we use to bypass the security stuff.

&#x20;

There's also tomcat version 8.5.5.

!\[\[60\_Kotarak\_image003.png]]

We take note of these.

&#x20;

For now, let's scan the ports 8080 and 60000 because they are the ones that are HTTP.

Jserv is a proxy that would basically feed the back-end server the requests that are sent to it.

&#x20;

While the scan is ongoing, let's take a look at the web page.

!\[\[60\_Kotarak\_image004.png]]

Interesting, this is a private web browser.

I opened up a HTTP server and tested it.

!\[\[60\_Kotarak\_image005.png]]

&#x20;

!\[\[60\_Kotarak\_image006.png]]

Works.

&#x20;

So this serves as a sort of file upload, but where do the files go?

I ran a directory scan on it, which told me that this was hosted on PHP.

!\[\[60\_Kotarak\_image007.png]]

&#x20;

The web page seems to load url.php with a specified path, and this would basically grant us the path it searches. Perhaps we could try to upload a malicious file onto this.

&#x20;

I checked out the info.php file for anything that was of use.

When trying to upload a reverse shell, this does not seem to execute. So this cannot be an upload vulnerability.

&#x20;

!\[\[60\_Kotarak\_image008.png]]

&#x20;

I know that this is located within the /var/www/html file, so I wondered if I was able to browse anything else other than that.

!\[\[60\_Kotarak\_image009.png]]

Well, this didn't work as planned.

Anyways, since this had loaded within the url.php, I wondered what other ports were open basically.

&#x20;

So I ran an intruder. To test all ports.

&#x20;

!\[\[60\_Kotarak\_image0010.png]]

While that was running, I knew that there was a tomcat server hosted, and that I was unable to view it. However, I wondered if I was able to access the URL for the tomcat-users.xml file, which contains the password and stuff for the engine.

&#x20;

Now I noticed something weird, there were different lengths of stuff.

!\[\[60\_Kotarak\_image0011.png]]

This means that it is able to poll the different ports from the web server there, so I left it running for a long time.

I tried accessing the tomcat users file from the url.php.

Then I noticed this.

!\[\[60\_Kotarak\_image0012.png]]

More and more ports with differing lengths.

&#x20;

&#x20;

!\[\[60\_Kotarak\_image0013.png]]

I also found this login for tomcat here.

!\[\[60\_Kotarak\_image0014.png]]

&#x20;

!\[\[60\_Kotarak\_image0015.png]]

The default credentials do not work.

We will keep this in mind for now, anyways there looks to be no way to access the tomcat-users.xml file within the browser. Burp Intruder was also taking way too long, so I wanted to use a curl script instead.

&#x20;

I know that this web browser seems to display nothing for things that it cannot find.

&#x20;

&#x20;

!\[\[60\_Kotarak\_image0016.png]]

So I wanted to see if I could ping all ports on this to get me what I want.

I took this script from another writeup I saw.

I left this running,

!\[\[60\_Kotarak\_image0017.png]]

Shellcheck is wonderful for vetting my scripts.

!\[\[60\_Kotarak\_image0018.png]]

This would print out the ports that matter, yes I could have made it easier but lazy...

!\[\[60\_Kotarak\_image0019.png]]

There seems to be a login page here, whereby we can log in.

&#x20;

!\[\[60\_Kotarak\_image0020.png]]

I have no credentials yet, so let's just ignore this one for now.

&#x20;

Seems that port 888 also produced something.

!\[\[60\_Kotarak\_image0021.png]]

&#x20;

Let's head there.

!\[\[60\_Kotarak\_image0022.png]]

&#x20;

There were these files here, but I could not access them for now, I had to append the ?doc at the back of the page finder on port 60000.

&#x20;

!\[\[60\_Kotarak\_image0023.png]]

&#x20;

Hmm, how do I access these files?

I viewed the page source, and noticed there were different hrefs being used.

!\[\[60\_Kotarak\_image0024.png]]

?nav seems to be used over? doc.

&#x20;

I tried changing the url to feature 10.10.10.55 instead of 127.0.0.1.

I curled the rest of it.

&#x20;

This returned me the password and stuff for tomcat from the backup folder.

!\[\[60\_Kotarak\_image0025.png]]

&#x20;

!\[\[60\_Kotarak\_image0026.png]]

Cool! Now we can log in easily and prepare that war file we always need.

!\[\[60\_Kotarak\_image0027.png]]

Generate a quick war file using msfvenom, and open a listener port.

!\[\[60\_Kotarak\_image0028.png]]

Upload this and execute.

!\[\[60\_Kotarak\_image0029.png]]

TTY shell it to stabilise and now we have gained the foothold into the machine.

&#x20;

Interestingly, there were backups or something within the tomcat user folder.

!\[\[60\_Kotarak\_image0030.png]]

Now, there were two folders here with some kind of backups, so let's transfer it back to our home directory.

&#x20;

!\[\[60\_Kotarak\_image0031.png]]

Seems that the .dit file is some kind of MS Windows registry file, so we perhaps could dump it out later.

&#x20;

Use netcat to transfer this back to our machine.

&#x20;

Now, let's try to see what we can dump from this file.

Turns out because it is an NTDS grab, we can use secretsdump.py to get out whatever is inside.

&#x20;

Anyways, when we transfer both of them and run secretsdump, we can get the hashes.

{width="9.53125in" height="3.3125in"}

There were a lot of hashes, but I just took note of the Administrator and atanas, which are our user and root respectively.

&#x20;

The admin password is f16tomcat!

!\[\[60\_Kotarak\_image0033.png]]

The user atanas password is Password123!

!\[\[60\_Kotarak\_image0034.png]]

&#x20;

So the password for atanas is the Administrator password.

!\[\[60\_Kotarak\_image0035.png]]

Interestingly, the root directory can be accessed by atanas.

!\[\[60\_Kotarak\_image0036.png]]

And the flag is not here...hmm..

&#x20;

So turns out there's another server present within the machine.

{width="11.729166666666666in" height="1.15625in"}

This app.log hints that it's somewhere.

&#x20;

And there's an archive to look through.

&#x20;

So there's another server on that IP address. Let's try to ping it.

!\[\[60\_Kotarak\_image0038.png]]

This works, now we just need to figure a way out to get into that machine.

I noticed there was a version in the wget there, and I wondered if that was something I had to use.

!\[\[60\_Kotarak\_image0039.png]]

Looks quite relevant, so I have it a shot.

Reading the exploit, this seems to ask us to host an FTP server and then transfer that stuff over.

I took the python script and changed it accordingly.

I took a look at my IP to the other host that was there.

!\[\[60\_Kotarak\_image0040.png]]

Seems that we have to use 10.0.3.1, as it's the IP that can talk to the other server.

We can then edit the script based on what kind of thing we would need.

&#x20;

There is a command injection here, and we can change that to execute a bash script for a reverse shell.

&#x20;

!\[\[60\_Kotarak\_image0041.png]]

So this script would execute a reverse shell back to us.

&#x20;

We then open up a FTP server of our own.

{width="14.135416666666666in" height="2.1666666666666665in"}

&#x20;

Then transfer it over and it should work.

!\[\[60\_Kotarak\_image0043.png]]

Then there's this error.

According to a walkthrough I looked through, turns out there was this permissions error when using ports above 1000, and we have to use authbind in order to bypass this.

Also remember to run this as atanas...

&#x20;

Run it like this:

!\[\[60\_Kotarak\_image0044.png]]

&#x20;

{width="13.59375in" height="2.2916666666666665in"}

Sooner or later, this would connect back to us (I hope).

&#x20;

{width="9.072916666666666in" height="0.6979166666666666in"}

This means that our crontab has been planted into the machine.

Now, this should eventually lead in a shell...

&#x20;

Until, I realised that I did not change the .wgetrc file.

!\[\[60\_Kotarak\_image0047.png]]

This would mean that nothing is being injected into crontabs.

&#x20;

I quickly put in a .wgetrc on both the target and victim.

&#x20;

!\[\[60\_Kotarak\_image0048.png]]

These are the contents as per the exploit, except I have redirected this to cron tab to make sure I get a root shell.

&#x20;

However, this did not work this for me...

&#x20;

Tried again and eventually...

!\[\[60\_Kotarak\_image0049.png]]

Finally...

&#x20;

Next round, it should serve me my shell.

!\[\[60\_Kotarak\_image0050.png]]

Grab the flag.

&#x20;

**\[Alternative Method]{.underline}**

Now, this is another method of which the exploit can happen.

This involves taking a look at the user and the groups he is in. I took a look at the walkthroughs that provided this pathway.

!\[\[60\_Kotarak\_image0051.png]]

So we are part of the disk group.

&#x20;

We then take a look at the disks to see how they are arranged.

{width="6.375in" height="2.2604166666666665in"}

&#x20;

As we can see, there are two disks that are under sda5 that are not supposed to be there.

&#x20;

!\[\[60\_Kotarak\_image0053.png]]

Take a look at this, and we can see where they are under.

&#x20;

This means that we can perhaps mount onto the file system from our machine.

&#x20;

We can then do this (from walkthrough):

!\[\[60\_Kotarak\_image0054.png]]

This would take a long ass time, but eventually we can get out the entire disk from that mount.

&#x20;

Unzip it and mount it, and then we would be able to see all the files within that server.

&#x20;

!\[\[60\_Kotarak\_image0055.png]]

&#x20;

!\[\[60\_Kotarak\_image0056.png]]

&#x20;

This is because the disk is located within the lxc file, something we are not allowed to access as atanas.

!\[\[60\_Kotarak\_image0057.png]]

Afterwards, we just need to look at the stuff within the file and get out the shell.

!\[\[60\_Kotarak\_image0058.png]]

By mounting it on our machine, we can access whatever file we want. Interesting method!

&#x20;

Overall, this was indeed a hard box. I'm glad that its been flagged as harder than OSCP.

&#x20;
