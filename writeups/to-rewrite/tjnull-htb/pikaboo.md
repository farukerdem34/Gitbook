# Pikaboo

Pikaboo

Tuesday, 1 March 2022

2:48 pm

This is an **Insane** Linux machine.

The target IP is 10.10.10.249

My IP is 10.10.16.5.

&#x20;

Enumeration!

!\[\[63\_Pikaboo\_image001.png]]

3 ports open.

Let's run a vuln script while looking at that ftp server for anonymous logins.

&#x20;

Anonymous login does not work.

For now, let's take a look at that website.

!\[\[63\_Pikaboo\_image002.png]]

&#x20;

Okay, the admin panel here requires some credentials.

!\[\[63\_Pikaboo\_image003.png]]

&#x20;

The website seems to be running on PHP as well, looking at the Pokatdex.

!\[\[63\_Pikaboo\_image004.png]]

When trying to view the pokemon, it says PokeAPI integration coming soon.

&#x20;

!\[\[63\_Pikaboo\_image005.png]]

Interesting. Let's take a look at Burpsuite and look at the requests that are being made.

There's an id parameter in the top that identifies which pokemon we are looking at. Currently we can see that there are 12 pokemon, wonder if there's more?

&#x20;

Inspecting the page, we can find this.

!\[\[63\_Pikaboo\_image006.png]]

There's a pokeapi.php file here...hmm..

&#x20;

Anyways, the detailed nmap scan came back at around this timing.

!\[\[63\_Pikaboo\_image007.png]]

I ran a directory scan on this machine just to check on stuff.

!\[\[63\_Pikaboo\_image008.png]]

Lots of directories pointing to the same place.

&#x20;

I began my search of pokeapi vulnerabilities but did not find much there.

Interestingly, this is the error I get when I key in the wrong credentials.

&#x20;

!\[\[63\_Pikaboo\_image009.png]]

Port 81? This would mean that the admin channel is being served on that port instead.

&#x20;

So this means that a reverse proxy is being used here.

&#x20;

I found some exploits regarding this.

!\[\[63\_Pikaboo\_image0010.png]]

&#x20;

This basically changes the way the web server reads the file we want to go to.

Understanding that this is an Apache server and we append .. It means nothing, I wondered if I was able to perhaps bypass the admin login.

&#x20;

This worked!

!\[\[63\_Pikaboo\_image0011.png]]

&#x20;

I was able to view more than one directory, and server-status was a code 200? Weird so I investigated as normally, we cannot access this directory.

!\[\[63\_Pikaboo\_image0012.png]]

I was able to view this and see some of the things that were used.

Everytime I refreshed this page, it seems that the links were changing, so someone is currently using this besides me?

There's this admin\_staging directory, and I tried accessing it as how I accessed the server-status directory.

&#x20;

This works, and I managed to get to the administrator panel.

!\[\[63\_Pikaboo\_image0013.png]]

&#x20;

Now, let's try to find an RCE somewhere.

I looked around to try and find some outlet of which I could perhaps inject some code in.

!\[\[63\_Pikaboo\_image0014.png]]

So there's this page parameter, and I was wondering if there was some kind of directory traversal attack that would be possible.

I used wfuzz to check on what is possible.

&#x20;

Used seclists and the LFI column to see if I can find anything.

&#x20;

Interestingly, this appeared within the folder.

!\[\[63\_Pikaboo\_image0015.png]]

&#x20;

!\[\[63\_Pikaboo\_image0016.png]]

Time to check these out.

The most interesting was the second, as you can see the length is actually different. This would mean that something actually appeared on the page.

Alright, and we do have something.

{width="6.8125in" height="1.65625in"}

It seems to mask the password...But I did find a user that works with this.

{width="5.770833333333333in" height="1.15625in"}

There's no password...But I do have a user!

&#x20;

How do I proceed from here?

Interestingly, this is where I was stumped for really long.

&#x20;

How do I proceed without the password?

I didn't want to DDOS the server using this.

!\[\[63\_Pikaboo\_image0019.png]]

&#x20;

Well, we d

&#x20;

We know that the logs are displayed within a PHP file on the website, so let's start from there and begin our enumeration.

So let's try do do some PHP injection into that file.

&#x20;

Googled around and determined that it was possible for us to execute PHP code using the Name parameter of the FTP connection.

Essentially, this would break me out of the log file within the web server, of which it is stored. Then, I can execute arbitrary code.

&#x20;

Tested out my little trick, and got a proof of concept.

!\[\[63\_Pikaboo\_image0020.png]]

This is executed by viewing the log file of the VSFTPD server.

&#x20;

Honestly, pretty stupid...

&#x20;

Anyways, we can replace my code there with a simple reverse shell and execute it again.

&#x20;

!\[\[63\_Pikaboo\_image0021.png]]

&#x20;

!\[\[63\_Pikaboo\_image0022.png]]

&#x20;

!\[\[63\_Pikaboo\_image0023.png]]

I'm in!

&#x20;

We can grab the user flag from this.

&#x20;

!\[\[63\_Pikaboo\_image0024.png]]

Right, now time to privilege escalate.

&#x20;

Downloaded linpeas over and found this.

!\[\[63\_Pikaboo\_image0025.png]]

&#x20;

!\[\[63\_Pikaboo\_image0026.png]]

&#x20;

This seems to be a job, whereby everytime something is uploaded with a .csv file within /srv/ftp folder, it would execute this script.

This would basically remove it(?) and this would be able to give us a shell perhaps.

&#x20;

Let's check out that folder within the ftp file.

&#x20;

!\[\[63\_Pikaboo\_image0027.png]]

This is a perl script, and it's quite long.

&#x20;

At the end, there is this being executed.

{width="7.395833333333333in" height="7.09375in"}

This is checking whether there are any .csv files being placed into that folder. So we can see that in this portion:

&#x20;

We can see that this is taking the file name right here, and then it parses it as a command.

!\[\[63\_Pikaboo\_image0029.png]]

This would be our command injection.

&#x20;

Now the question is this, how do we inject commands into perl with a file name?

!\[\[63\_Pikaboo\_image0030.png]]

This little text here helps!

Since this script is run by root, then this should basically interpret the filename as a command.

&#x20;

So we have to create a .csv file with a name like | \<insert bash script here>.csv.

&#x20;

This must be the exploit.

&#x20;

Now, we need to find a way to upload to that directory, and I suspect that this has something to do with the FTP server.

&#x20;

Let's go password hunting.

&#x20;

I first checked the opt file, which had a pokeapi file within it.

!\[\[63\_Pikaboo\_image0031.png]]

Let's check the file to find stuff.

&#x20;

!\[\[63\_Pikaboo\_image0032.png]]

Looking through all these files here.

Within the settings.py, I found this cool little secret\_key.

&#x20;

!\[\[63\_Pikaboo\_image0033.png]]

As well as a hint towards ldap searching.

&#x20;

!\[\[63\_Pikaboo\_image0034.png]]

Might need to use this!

This brings me to something called ldapsearch.

Around here, I was unable to get it to work so I used a walkthrough in the end...

&#x20;

Eventually, I managed to get ldapsearch to work and dump out all the information.

&#x20;

!\[\[63\_Pikaboo\_image0035.png]]

Saw this.

&#x20;

!\[\[63\_Pikaboo\_image0036.png]]

The ftp password was the most interesting!

This decrypted to this

"\_G0tT4\_C4tcH\_'3m\_4lL!\_"

&#x20;

From here, let's try logging into the ftp server and placing files within that .csv folder.

!\[\[63\_Pikaboo\_image0037.png]]

Once in, I figured out that we were actually already within the directory I needed to place files in, which was...convenient. Except that all the file names had their .csv removed.

&#x20;

Based on the dates on the ftp server, looks like the file called 'versions' is the most updated.

!\[\[63\_Pikaboo\_image0038.png]]

Let's go there!

&#x20;

Now, we should only need to put some fake .csv file with nothing in it, just the filename being the command we want to execute. I put this.

&#x20;

!\[\[63\_Pikaboo\_image0039.png]]

Opened up a listener port and just stood by waiting.

I was unsure if this would work out well, and I tried to remove more of those special commands and stuff.

&#x20;

There was no netcat installed on the machine, so that could not have been used.

&#x20;

Waited around for a few minutes but this did not work...

I took note that there were special characters that needed to be escaped properly, hence I tried again using this one.

&#x20;

!\[\[63\_Pikaboo\_image0040.png]]

Eventually...

!\[\[63\_Pikaboo\_image0041.png]]

Here we go.

&#x20;

I found this quite hard, but was glad to have identified the correct errors within the perl script.

&#x20;

&#x20;
