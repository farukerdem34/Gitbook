# Quick

**Quick**

Wednesday, 9 March 2022

9:25 pm

This is an **Insane** Linux machine.

The target IP is 10.10.10.186.

My IP is 10.10.16.9.

&#x20;

Enumeration time! Expect a lot of fun for this.

!\[\[74\_Quick\_image001.png]]

&#x20;

!\[\[74\_Quick\_image002.png]]

&#x20;

Website:

!\[\[74\_Quick\_image003.png]]

&#x20;

HTTPS website right there... And this below:

!\[\[74\_Quick\_image004.png]]

&#x20;

!\[\[74\_Quick\_image005.png]]

This seems to be a PHP website.

!\[\[74\_Quick\_image006.png]]

&#x20;

Let's go to that HTTPS server, but make sure to add portal.quick.htb to our hosts file first.

That does not work however. Welp, perhaps a rabbit hole.

&#x20;

Looking at the page, there's this login.php file as well:

!\[\[74\_Quick\_image007.png]]

Ticketing system, perhaps Kerberos authentication?

&#x20;

Ran a directory enumeration in the background, so while that is running let's take a look at the website itself.

&#x20;

&#x20;

&#x20;

!\[\[74\_Quick\_image008.png]]

I still ran a directory scan against this website.

Seems to be an identical website to the other one.

&#x20;

!\[\[74\_Quick\_image009.png]]

&#x20;

There's one directory there with a db.php file located. Perhaps we can examine the request in Burp to see what we can do against it.

&#x20;

When we submit a wrong password and username, it gives me this:

!\[\[74\_Quick\_image0010.png]]

&#x20;

Sqlmap does not give me anything either.

I didn't notice any boolean condition, besides the fact that when we enter a wrong email or something it does not proceed with the script there.

&#x20;

I was a bit lost, and I knew that the https server that I could not access had to lead me to somewhere.

Perhaps this was using HTTP/2 or something, and it does not have a backend server to handle these older requests.

&#x20;

Googling around led me to [HTTP/3](https://github.com/curl/curl/blob/master/docs/HTTP3.md), which is basically this new technology called QUIC.

There's [this](https://gist.github.com/syndrowm/62d8643c0651e038616ec3d5fafc96bb) script which would install http3 onto curl and allow us to access this supposed website.

&#x20;

All of this is a trial, and if I don't know I'll move on with something else.

&#x20;

Skipped the installation part as it was being a headache.

Read a walkthrough and confirmed that I was on the right path with HTTP/3.

Turns out, we just needed to read through the PDFs in the HTTP/3 server. This would give us a password of

Quick4cc3\$$

&#x20;

The email was also on the HTTP/3 page.

!\[\[74\_Quick\_image0011.png]]

&#x20;

!\[\[74\_Quick\_image0012.png]]

Tried all of these names followed by quick.htb, and elisa@wink.co.uk works.

&#x20;

Logged in and found the ticket tracking system:

!\[\[74\_Quick\_image0013.png]]

I was able to raise tickets through this:

!\[\[74\_Quick\_image0014.png]]

&#x20;

!\[\[74\_Quick\_image0015.png]]

When submitted, this gave me a TKT name:

&#x20;

!\[\[74\_Quick\_image0016.png]]

&#x20;

For some reason, I was unable to get my display to show up on the search bar.

Looking at the source code, it seems to execute some JS:

!\[\[74\_Quick\_image0017.png]]

&#x20;

From here, we have to determine how we are going to get that out.

&#x20;

Back to Burpsuite!

&#x20;

But first, I remember this:

!\[\[74\_Quick\_image0018.png]]

&#x20;

This was something interesting I saw.

When searching for Esigate exploits, it brought me to ESI Injection, which is the injection of ESI tags in pages to fool the cache proxy.

Let's do some tests for this.

&#x20;

[This](https://www.gosecure.net/blog/2019/05/02/esi-injection-part-2-abusing-specific-implementations/) blog showed us a bit about RCE through XSLT. This would include appending the following script to the end of some request.

!\[\[74\_Quick\_image0019.png]]

Now all we needed was some kind of script, which it also provided.

&#x20;

!\[\[74\_Quick\_image0020.png]]

&#x20;

!\[\[74\_Quick\_image0021.png]]

Let's test this and host this on a python web server.

This is an .xsl file that we have to use.

&#x20;

From here, let's try to upload this somehow.

&#x20;

Noticed that there was a POST request when we used /ticket.php to submit tickets to the web server.

!\[\[74\_Quick\_image0022.png]]

Let's append our script and open a Python server at the same time.

!\[\[74\_Quick\_image0023.png]]

This worked!

&#x20;

{width="4.583333333333333in" height="1.4583333333333333in"}

Ignore my typo, and let's try again.

Open an ICMP listener too.

Changed the name of this file to c.xsl.

!\[\[74\_Quick\_image0025.png]]

&#x20;

{width="9.4375in" height="2.1041666666666665in"}

We have RCE!

&#x20;

!\[\[74\_Quick\_image0027.png]]

Now all that's left is just to get a reverse shell on this thing.

&#x20;

This does not work, and it seems that no reverse shells work on this machine.

&#x20;

So we have to do it the other way, which is create the shell on our machine and then upload it onto the server, followed by executing the script.

This was easy enough.

&#x20;

Create a quick rev.sh:

!\[\[74\_Quick\_image0028.png]]

&#x20;

From here, we just need to create another .xsl with the command to download this into the /tmp directory as rev.sh

!\[\[74\_Quick\_image0029.png]]

This would download the script, then create another one to execute the script.

&#x20;

!\[\[74\_Quick\_image0030.png]]

&#x20;

Keep in mind the names, which are wget and exe for well, wget and execute.

Upload the wget.xsl and this would download the script.

!\[\[74\_Quick\_image0031.png]]

Start a listener port on port 4444 and then send the exe.xsl to execute this script. Then realise you shouldn't have ./tmp/rev.sh with the period in front.

And also not port 4444 because firewalls are probably blocking this. Jesus.

&#x20;

Eventually....

!\[\[74\_Quick\_image0032.png]]

&#x20;

Grab the user flag and let's press on.

Seems that there are 2 users on this box:

!\[\[74\_Quick\_image0033.png]]

Now, we should aim to get into srvadm.

&#x20;

There's this little hint that we might me within a container in the /opt directory.

!\[\[74\_Quick\_image0034.png]]

&#x20;

I remembered that there was a db.php within the server, and this has some form of credentials for us.

!\[\[74\_Quick\_image0035.png]]

Su does not work, so let's look at mysql or something.

&#x20;

!\[\[74\_Quick\_image0036.png]]

The database we are interested in is quick:

!\[\[74\_Quick\_image0037.png]]

&#x20;

Looking at the quick, there are 3 tables, one of which is called users with a password!

!\[\[74\_Quick\_image0038.png]]

This hash seems salted or something, else it would be too easy.

&#x20;

Looking at the login.php, we can see this little script:

{width="11.041666666666666in" height="5.4375in"}

Seems that our hash right there is indeed salted, and I can see that it crypts the password with $password, which would

&#x20;

This would require a quick python script or something.

Getting my hashes out, it seems that we have to try and figure out a way to get us the hash we got.

&#x20;

Now, based on the crypt, it seems that it takes the password and then crypts it, followed by hashing it.

!\[\[74\_Quick\_image0040.png]]

&#x20;

Cool!

But since we can alter the database, we can just change the password instead, save the effort of figuring out the crypt and stuff.

!\[\[74\_Quick\_image0041.png]]

Now, we can log into that account using the same password as before.

&#x20;

This does not grant us the stuff that we want however. I then ran a netstat to see the ports, and also created a .ssh directory and echoed in my ssh key to upgrade my shell.

&#x20;

!\[\[74\_Quick\_image0042.png]]

There's a port 80 there. Let's port tunnel to it.

!\[\[74\_Quick\_image0043.png]]

Nothing yet, but I did find this within the apache2 files:

!\[\[74\_Quick\_image0044.png]]

Let's head to it.

&#x20;

This always seem to redirect me back to the regular home page however.

&#x20;

!\[\[74\_Quick\_image0045.png]]

&#x20;

I was supposed to be getting something else, but clearly I'm wrong.

&#x20;

I looked at a guide for the rest as I was a bit lost.

&#x20;

**This is the start of the walkthrough.**

&#x20;

Turns out we are supposed to be able to go into the printer webpage, but we cannot.

!\[\[74\_Quick\_image0046.png]]

From here, we can create new jobs.

&#x20;

!\[\[74\_Quick\_image0047.png]]

We are able to put our IP address within that, and from there we are able to get a NC connection.

!\[\[74\_Quick\_image0048.png]]

&#x20;

!\[\[74\_Quick\_image0049.png]]

&#x20;

Looking at the job.php code, we can see this:

{width="10.21875in" height="7.239583333333333in"}

So this would basically create a file with a timestamp, and get the contents of this file. So from here, we can redirect this file, as we can see it does not check for validation of the file here.

&#x20;

There's a .ssh file within the srvadmin files too.

&#x20;

When we execute a job, there are files created within the /job directory. From there, we are able to just create an infinite bash loop which would wait for the file to be generated, then generate a quick symlink to the ssh private key.

{width="4.03125in" height="2.0104166666666665in"}

We execute this within the job file that we create for ourselves.

&#x20;

Then execute another job, and out comes the private key.

!\[\[74\_Quick\_image0052.png]]

&#x20;

**This is the end of the walkthrough help**.

&#x20;

SSH into srvadm and we are in. From here, we can do an ls -la.

!\[\[74\_Quick\_image0053.png]]

There is not much left on the webserver, so we can investigate every single one of these files.

&#x20;

Within the .cache file, which took a long time to decide to go into, there's this conf.d file here from a walkthrough, which is a pretty stupid method of hiding a password.

!\[\[74\_Quick\_image0054.png]]

Within this, there are more configuration files.

!\[\[74\_Quick\_image0055.png]]

From this, we can see some conf files.

&#x20;

Investigating this, we can see that there is a printerv3 web server here, and what looks to be another password.

!\[\[74\_Quick\_image0056.png]]

Decrypted, this gives us a password looks like:

&#x20;

!\[\[74\_Quick\_image0057.png]]

&#x20;

This is indeed the root password. Su and get the flag!

!\[\[74\_Quick\_image0058.png]]

&#x20;

This was an exhausting box. The web server not working for me was a real bummer, but even if I were to have it I probably would have not been able to figure out the symlink.

&#x20;
