# Deployer

Deployer

Sunday, June 26, 2022

2:58 PM

Nmap scan:

!\[\[62\_Deployer\_image001.png]]

&#x20;

!\[\[62\_Deployer\_image002.png]]

!\[\[62\_Deployer\_image003.png]]

&#x20;

&#x20;

Port 80:

!\[\[62\_Deployer\_image004.png]]

&#x20;

Add to hosts file and press on.

!\[\[62\_Deployer\_image005.png]]

&#x20;

Not much here.

&#x20;

Port 21:

!\[\[62\_Deployer\_image006.png]]

&#x20;

We can see that there's another dev portal there, and it seems to contain PHP files.

!\[\[62\_Deployer\_image007.png]]

&#x20;

And some lfi-prev.html? This seems to be the website we want to access.

It seems that there is also a file within the /html file, which just refreshes and forces the user to visit the [http://deployer.off](http://deployer.off) website.

!\[\[62\_Deployer\_image008.png]]

&#x20;

However, because of the configuration of the apache server, this isn't possible because the document root is always going to be visiting that website.

{width="8.729166666666666in" height="3.1041666666666665in"}

&#x20;

From looking at the contact.php page of the /dev directory, we can see where the LFI comes in:

!\[\[62\_Deployer\_image0010.png]]

&#x20;

I downloaded the entire page and just sent loaded it on my machine:

!\[\[62\_Deployer\_image0011.png]]

&#x20;

Straightaway, we can see this. There's a unserializing of input by the user, and there's very little...verification behind it.

&#x20;

Within the FTP files, we can find some weird .conf files here.

!\[\[62\_Deployer\_image0012.png]]

&#x20;

We can see how there are 3, and viewing 002 reveals this:

{width="5.21875in" height="3.3229166666666665in"}

&#x20;

Now we can access that hidden website.

And then from there, we can begin to experiment with our LFI.

&#x20;

LFI:

&#x20;

!\[\[62\_Deployer\_image0014.png]]

&#x20;

This is the function we are dealing with, and we can see how it is unserialised further. This is the code that we have to exploit.

We can follow this function.

* Firstly, it checks for there being a POST parameter called page, and if it's not set, then it would begin the LFI portion, checking for '..'
* \_\_wakeup() is called when the object is deserialized. This would mean that
  * If the '..' was detected, then it would trigger the alert below.
  * If there isn't an LFI present, it would include the page that has the /var/www/dev thing appended in front of it.
* If we were to send a POST request, then it would take the value of that and unseralize it to do...something.

&#x20;

Now, for some weird reason, it seems that we are unable to escape the portion where the page is at. Meaning that we are stuck inside this /dev directory.

I think it's because we are attached to the /var/www/dev portion of the code.

&#x20;

Now, we can see that whenever something is deserialised, it would cause the wakeup() function to activate, as it's a PHP magic method which would basically mean that it would include the variables we have included within the deserialisation.

&#x20;

So to exploit this, we can do it this way:

!\[\[62\_Deployer\_image0015.png]]

&#x20;

We can create a quick exploit code here, and then create an object.

!\[\[62\_Deployer\_image0016.png]]

&#x20;

Then we can send this as a POST parameter with the name page:

!\[\[62\_Deployer\_image0017.png]]

&#x20;

Success! So we can read files now, but that does not mean we can execute code yet. This is due to:

* There isn't an eval function anywhere
* I'm pretty sure that we need to read some hidden file again, which is very annoying.
* Either that or we upload a file...

&#x20;

I thought of a great idea, why not include some form of php code in the FTP file that we are allowed to upload to?

We just need to find out where this ftp folder really is.

&#x20;

We can read the vsftpd conf files.

!\[\[62\_Deployer\_image0018.png]]

&#x20;

Afterwards, we can gain a shell:

{width="8.979166666666666in" height="0.9791666666666666in"}

&#x20;

!\[\[62\_Deployer\_image0020.png]]

&#x20;

{width="8.90625in" height="2.2291666666666665in"}

&#x20;

Flag:

!\[\[62\_Deployer\_image0022.png]]

&#x20;

PE:

Within /opt there's this id\_rsa.bak.

!\[\[62\_Deployer\_image0023.png]]

&#x20;

It's only readable by root, which is interesting.

&#x20;

Checking out on the docker images, we can see there a few things mounted.

!\[\[62\_Deployer\_image0024.png]]

&#x20;

The interesting part is that this a whole image of the current system on another system. Perhaps we have to mount this or something.

!\[\[62\_Deployer\_image0025.png]]

&#x20;

When trying to sudo, we can see this.

!\[\[62\_Deployer\_image0026.png]]

&#x20;

I'm not sure what this means, but it definitely is weird. Why is it setting the gid and id as -1?

So the a value of -1 means that there was no change.

&#x20;

This would mean that well, sudo is unable to change the stuff that we want to see.

The shell that we are running on right now is a product of a separate uid and gid that is on the VM, meaning we cannot really access the real GID and UID because we are on a shell.

&#x20;

This is running on apache2-mpm-itk, which is just a separate shell.

&#x20;

To bypass this, we can just SSH into the user instead.

&#x20;

Adding our key:

!\[\[62\_Deployer\_image0027.png]]

&#x20;

!\[\[62\_Deployer\_image0028.png]]

&#x20;

And now, we can view stuff like this.

&#x20;

!\[\[62\_Deployer\_image0029.png]]

&#x20;

We can see our sudo privileges and also what images are present.

So basically, we can build a new image.

&#x20;

Docker build takes a Dockerfile to do, and we can make that easily.

Dockerfile:

!\[\[62\_Deployer\_image0030.png]]

&#x20;

!\[\[62\_Deployer\_image0031.png]]

&#x20;

{width="7.4375in" height="1.6875in"}

&#x20;

With this key, we can SSH in as root.

&#x20;

Flag:

!\[\[62\_Deployer\_image0033.png]]

&#x20;

78fef41b4ed702d509a7d45f664831c8

&#x20;

&#x20;
