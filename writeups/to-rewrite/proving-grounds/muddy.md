# Muddy

Muddy

Wednesday, 30 March 2022

5:59 pm

A Linux machine with these ports.

&#x20;

!\[\[44\_Muddy \_image001.png]]

&#x20;

Port 80 just shows a wordpress website.

!\[\[44\_Muddy \_image002.png]]

&#x20;

!\[\[44\_Muddy \_image003.png]]

&#x20;

There doesn't seem to be much that we can do about this website as of now.

&#x20;

!\[\[44\_Muddy \_image004.png]]

&#x20;

SMTP seems to support loads of commands.

Port 8888 shows me this.

!\[\[44\_Muddy \_image005.png]]

&#x20;

Ladon services...

!\[\[44\_Muddy \_image006.png]]

There are exploits for this, so let's check it out.

&#x20;

It seems that we are able to do XXE and be able to read files on the host device.

When testing this out, we can see that it kind of works.

!\[\[44\_Muddy \_image007.png]]

&#x20;

There are errors being generated and that's kind of a good sign.

!\[\[44\_Muddy \_image008.png]]

Looking at the code present on the website, we see that this uses checkout instead of sayHello.

&#x20;

Also, we can take note that it uses a dispatcher.py instead of a helloservice.py within the code there.

&#x20;

After a bit of testing, I got it to work.

&#x20;

!\[\[44\_Muddy \_image009.png]]

WE can now view the passwd file. We can als osee that there is a user called ian wihtin the machine.

!\[\[44\_Muddy \_image0010.png]]

&#x20;

Did a dirbuster on the port 80 to find out more information about the port 80 web page.

I cannot seem to find the wordpress config file, and hence cannot really log in to the WP website. Not that there's any exploits I can do there anyway.

&#x20;

!\[\[44\_Muddy \_image0011.png]]

The gobuster reveals there is a webdav page here.

When trying to access it, it gives us a username and password to enter.

&#x20;

!\[\[44\_Muddy \_image0012.png]]

&#x20;

We have a local file reader, so we should aim to find this one file.

&#x20;

!\[\[44\_Muddy \_image0013.png]]

&#x20;

!\[\[44\_Muddy \_image0014.png]]

We can use the file reader vulnerability to read this password from the /var/www/html/webdav/passwd.dav folder.

!\[\[44\_Muddy \_image0015.png]]

John reveals the password.

&#x20;

Now that we can login, we can basically upload shells onto the website.

&#x20;

Ran a davtest to check which files would execute, and it seems PHP works best.

!\[\[44\_Muddy \_image0016.png]]

We can upload a cmd.php shell this way.

!\[\[44\_Muddy \_image0017.png]]

From here, we have RCE.

&#x20;

!\[\[44\_Muddy \_image0018.png]]

Get a reverse shell through nc.

&#x20;

!\[\[44\_Muddy \_image0019.png]]

&#x20;

!\[\[44\_Muddy \_image0020.png]]

Then stabilise the shell.

It seems that the user flag was not in the user ian's directory.

&#x20;

!\[\[44\_Muddy \_image0021.png]]

Right, time to PE.

!\[\[44\_Muddy \_image0022.png]]

There are some database creds here.

&#x20;

Got linpeas onto the machine to find more PE vectors.

!\[\[44\_Muddy \_image0023.png]]

It seems that we have this in our path.

&#x20;

!\[\[44\_Muddy \_image0024.png]]

There's also this part here, but I don't think we can reboot the machine.

&#x20;

What this means to be is that netstat seems to be running on the machine every minute or so, and we have a writeable path. When we inject a script that is called netstat, it will be run as root. From there, we can gain a reverse shell.

From here, we can create a netstat shell inside the /dev/shm directory and include the following:

!\[\[44\_Muddy \_image0025.png]]

Wait for a minute or so, and we will eventually get a root shell back.

&#x20;

!\[\[44\_Muddy \_image0026.png]]

&#x20;

!\[\[44\_Muddy \_image0027.png]]

&#x20;
