# Escape

Escape

Sunday, July 10, 2022

3:36 PM

Nmap scan:

!\[\[79\_Escape \_image001.png]]

&#x20;

!\[\[79\_Escape \_image002.png]]

&#x20;

Port 80:

All we see is this one page.

!\[\[79\_Escape \_image003.png]]

&#x20;

There is one jail.jpg file, of which we can download the image and then view the file.

Port 8080 contains another of the same thing. It's another jail cell. When we download the images and view them using exiftool and strings, we get nothing.

&#x20;

Anyways, we can edit this CSS and make the website point to different locations. Perhaps we should point it towards a file or something.

&#x20;

!\[\[79\_Escape \_image004.png]]

&#x20;

!\[\[79\_Escape \_image005.png]]

&#x20;

This works, so in this case, perhaps we can make the file read its own stuff and display it on screen with some SSRF.

&#x20;

Through this, we can make the web page load whatever we want.

!\[\[79\_Escape \_image006.png]]

&#x20;

In this case, we can construct some HTML frames to make a connection and be able to read files through the system.

&#x20;

!\[\[79\_Escape \_image007.png]]

&#x20;

Above didn't work, so I tried again.

&#x20;

!\[\[79\_Escape \_image008.png]]

&#x20;

Trying to load files using the file:/// wrapper is not useful.

!\[\[79\_Escape \_image009.png]]

&#x20;

There are CSS Injection attacks that steal tokens, but I'm unsure if that is the way to go for this one.

&#x20;

XSS doesn't work. As such, let's try to brute force directories within this server and find some hidden directories.

We can try testing with SSRF through HTTP headers to perhaps find more directories present on the server.

&#x20;

On port 8080, this was found.

!\[\[79\_Escape \_image0010.png]]

&#x20;

!\[\[79\_Escape \_image0011.png]]

!\[\[79\_Escape \_image0012.png]]

&#x20;

When we try to upload something, it responds with this.

!\[\[79\_Escape \_image0013.png]]

&#x20;

What we can do is firstly, get a PHP reverse shell with a GIF header, then change the parameters for upload to this.

!\[\[79\_Escape \_image0014.png]]

&#x20;

Then we can visit the /uploads/test.gif.php page and gain a shell as www-data.

&#x20;

{width="9.791666666666666in" height="2.78125in"}

&#x20;

Flag:

!\[\[79\_Escape \_image0016.png]]

&#x20;

This thing looks like a container to me.

&#x20;

When we view the /var/backups file, we can see this.

!\[\[79\_Escape \_image0017.png]]

&#x20;

There's an SNMPD.conf file.

We read it and within it, we find this password here.

!\[\[79\_Escape \_image0018.png]]

&#x20;

We can't login to root with this, but what we can do it simply check SNMP on UDP port 161.

&#x20;

We can begin to enumerate this.

!\[\[79\_Escape \_image0019.png]]

&#x20;

&#x20;

And we can indeed confirm we are in a docker container.

!\[\[79\_Escape \_image0020.png]]

&#x20;

What we can do is simple, we can see below the arbitrary execution of commands.

!\[\[79\_Escape \_image0021.png]]

&#x20;

!\[\[79\_Escape \_image0022.png]]

&#x20;

We can do something like this, and then see that it works.

{width="9.791666666666666in" height="1.9270833333333333in"}

&#x20;

Then, we just need to replace this with a shell.

Then we can replace the command with this.

!\[\[79\_Escape \_image0024.png]]

&#x20;

{width="9.854166666666666in" height="1.25in"}

&#x20;

{width="8.322916666666666in" height="2.2291666666666665in"}

&#x20;

We are now within the actual machine.

Here's the proper flag:

!\[\[79\_Escape \_image0027.png]]

&#x20;

PE:

I ran LinPEAS to enumerate for me.

I also ran pspy to see the processes being run and if there are any cronjobs to sabotage.

&#x20;

We could also try looking for some credentials and SSH in as the user 'tom'.

&#x20;

Interesting LinPEAS output:

!\[\[79\_Escape \_image0028.png]]

&#x20;

!\[\[79\_Escape \_image0029.png]]

&#x20;

!\[\[79\_Escape \_image0030.png]]

&#x20;

There was this unknown SUID binary for tom.

!\[\[79\_Escape \_image0031.png]]

&#x20;

Likely this is the next step.

&#x20;

Firstly, let's take a look at this binary through ltrace.

!\[\[79\_Escape \_image0032.png]]

Within this, we can see that this binary has the ability to do setuid and seteuid.

!\[\[79\_Escape \_image0033.png]]

&#x20;

Basically, this binary runs system commands as the other user (tom) and prints the output there.

&#x20;

When selecting option 99, we can see that this basically generates some kind of fault because it attempts to open a file that doesn't exist.

!\[\[79\_Escape \_image0034.png]]

&#x20;

When we run some of the commands, we can view that the lscpu command does not have a service path.

!\[\[79\_Escape \_image0035.png]]

&#x20;

Which is easy for us.

&#x20;

We just need to make it such that lscpu is a binary after exporting /tmp to be the first directory in PATH environment variable.

!\[\[79\_Escape \_image0036.png]]

&#x20;

!\[\[79\_Escape \_image0037.png]]

&#x20;

Executing this would result in a shell.

{width="7.208333333333333in" height="1.9375in"}

&#x20;

Now from here, we can check on the other binary within /opt, which was openssl with the capabilities set.

!\[\[79\_Escape \_image0039.png]]

&#x20;

It's likely that key.pem might be some SSH thing, and that only tom can execute openssl.

&#x20;

To abuse this, we likely need to do something like host a web server.

!\[\[79\_Escape \_image0040.png]]

&#x20;

This was on GTFOBins.

&#x20;

So let's just do that.

We can run the first command, and then run a server.

Then, we need to head into the main / directory and host a HTTP server with the following key and cert that we have generated.

&#x20;

(at this point, I just used the walkthrough because I was unsure of what I was doing)

&#x20;

!\[\[79\_Escape \_image0041.png]]

&#x20;

Then we just grab another shell using SSH (just make .ssh file and SSH using the below command):

!\[\[79\_Escape \_image0042.png]]

&#x20;

Afterwards, this simply means we are able to have a HTTPS server of the entire / directory.

!\[\[79\_Escape \_image0043.png]]

&#x20;

We can grab the private key and ssh in as root.

&#x20;

Flag:

!\[\[79\_Escape \_image0044.png]]

&#x20;

Interesting PE vector.

&#x20;
