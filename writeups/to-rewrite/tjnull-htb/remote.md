# Remote

Remote

Sunday, 20 February 2022

8:52 pm

This is a Windows machine.

The target IP is 10.10.10.180.

My IP is 10.10.16.9.

&#x20;

As usual. Take note this box seems to block ICMP echo requests, so make sure to append the -Pn flag to the nmap scan.

&#x20;

!\[\[47\_Remote\_image001.png]]

&#x20;

!\[\[47\_Remote\_image002.png]]

&#x20;

**Again, null sessions are not allowed on this machine.**

Saw the FTP server, and tried anonymous log in, which worked.

&#x20;

!\[\[47\_Remote\_image003.png]]

Unfortunately, there's nothing of interest in this.

&#x20;

Let's look at the web page.

!\[\[47\_Remote\_image004.png]]

Cool little website.

Too many directories present.

&#x20;

!\[\[47\_Remote\_image005.png]]

Going back to my nmap scan, I checked on the port 2049, which was an interesting port regarding stuff.

&#x20;

I was able to view more information regarding the website.

&#x20;

!\[\[47\_Remote\_image006.png]]

&#x20;

This involved using some nfs scripts, which would display stuff for us. Anyways we would need to mount something on this, in order to allow us to be able to view the files there successfully. We know the directory is site\_backups.

&#x20;

!\[\[47\_Remote\_image007.png]]

&#x20;

This indicates to us that we can indeed access this.

&#x20;

!\[\[47\_Remote\_image008.png]]

Now let's take a look at the stuff within here.

&#x20;

Googling a little bit, I found this .sdf file, which is something like a database file with information within it.

&#x20;

!\[\[47\_Remote\_image009.png]]

&#x20;

There's some form of hashes herte that we can decrypt and stuff.

Get this hash, and decrypt it right away.

&#x20;

{width="11.375in" height="4.114583333333333in"}

This is the password. But to what? I have not found anything to log in with yet.

&#x20;

So I know that the website runs on Umbraco, judging from the files that were present in the mount. Hence I googled and it seems that an admin page exists on /Umbraco.

&#x20;

!\[\[47\_Remote\_image0011.png]]

&#x20;

There needs to be an email somewhere, because admin does not work.

Tried admin@htb.local and it worked lol.

!\[\[47\_Remote\_image0012.png]]

&#x20;

Now, we just need to find some form of exploit that can RCE for us.

!\[\[47\_Remote\_image0013.png]]

Find the version here.

!\[\[47\_Remote\_image0014.png]]

&#x20;

Fantastic!

&#x20;

Now, let's get ourselves a shell somehow.

&#x20;

So we get the first script, and we change the payload accordinfgly.

&#x20;

!\[\[47\_Remote\_image0015.png]]

&#x20;

This would basically download a powershell script from us, and that will be our reverse shell going in.

So after doing this, just get some random nishang powershell reverse shell script, and host a HTTP server on our machine.

&#x20;

!\[\[47\_Remote\_image0016.png]]

&#x20;

!\[\[47\_Remote\_image0017.png]]

&#x20;

!\[\[47\_Remote\_image0018.png]]

We're in.

Grab the user flag from the Public directory, and now to PE.

I got powerup.ps1 to help enumerate a few things. According to hacktricks, I could also do an Invoke-AllChecks to check on the stuff that we perhaps abuse.

&#x20;

&#x20;

!\[\[47\_Remote\_image0019.png]]

This was one of them.

Unfortunately, nothing I did worked out well.

&#x20;

Moving on, this does not seem to be the solution.

I went into the Program Files, and found a TeamViewer file. This was not supposed to be here.

&#x20;

At this point, I kinda was not sure anymore, so I looked up a walkthrough.

&#x20;

Turns out TeamViewer is the correct file to be viewing, and this would result in us having to find the password, the most popular method seems to be using MSF to view the password somehow.

&#x20;

So turns out TeamViewer is indeed not supposed to be there, hence there are password to uncover from this. We can then use the MSF module to view it and then use psexec to log in.

&#x20;

{width="9.260416666666666in" height="3.9166666666666665in"}

&#x20;

!\[\[47\_Remote\_image0021.png]]

&#x20;

So what we actually had to do to get the password was more difficult, we had to actually get out the password in some encrypted form. This would give us the security key if we do the following.

&#x20;

!\[\[47\_Remote\_image0022.png]]

&#x20;

This would be done through taking a look at the MSF module that is doing this for us, where in this case it would be the TeamViewer code.

&#x20;

Now, from here, we are able to get out this string of numbers.

&#x20;

!\[\[47\_Remote\_image0023.png]]

After that, we can pass this string into the decrypt function of the device easily to get out the password.

&#x20;

{width="8.8125in" height="5.53125in"}

This would easily decrypt it and give us the password.

&#x20;

Afterwards, we can simply just log in using Win-RM.

&#x20;

My Windows skills aren't as good, need to practice more.
