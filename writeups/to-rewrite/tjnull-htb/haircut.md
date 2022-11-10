# Haircut

Haircut

Wednesday, 9 February 2022

10:35 am

This is a Linux machine.

My IP is 10.10.16.5

The target IP is 10.10.10.24.

&#x20;

Enumeration.

&#x20;

!\[\[28\_Haircut\_image001.png]]

&#x20;

SSH and HTTP here. Likely we need to get some password and SSH in perhaps.

&#x20;

Ran a directory enumeration against the website as well, in case.

Early signs show an /uploads category, perhaps we need to insert a PHP reverse shell.

!\[\[28\_Haircut\_image002.png]]

&#x20;

The website just shows this...

!\[\[28\_Haircut\_image003.png]]

Nikto picked on on test.html, which also contains nothing much but a Carrie Curl...

&#x20;

!\[\[28\_Haircut\_image004.png]]

&#x20;

Ran a more in-depth directory scan and found more stuff.

!\[\[28\_Haircut\_image005.png]]

&#x20;

There's an exposed.php, which brings us to this page.

!\[\[28\_Haircut\_image006.png]]

Played around with the thing, and it seems to be able to check out web pages and stuff. I tried putting in something funny and went to my own web page, after hosting a HTTP server.

&#x20;

It works.

!\[\[28\_Haircut\_image007.png]]

&#x20;

Now I was trying to see whether I could get some kind of command line or PHP shell. It does not seem to execute php code within the browser.

&#x20;

I intercepted the responses with burp, just to see whats' going on.

!\[\[28\_Haircut\_image008.png]]

&#x20;

When intercepting the response, I can see that this is hiding within the code. This means the website actually outputs it on the web page, so let's see if it can go elsewhere.

&#x20;

I know that the hint is curl, and I guessed that perhaps this machine is running curl on the exposed.php.

&#x20;

Curl has an option to output a file, using the flag -o. I know there's an uploads directory. So I tried my reverse php shell and outputted it to /var/www/html/uploads/shell.php.

&#x20;

Ran it, and I got a shell.

!\[\[28\_Haircut\_image009.png]]

&#x20;

In this case we are www-data. Grab the user flag while we are in.

&#x20;

Priv escalate time! Downloaded LinEnum.sh because it's the easiest to use. Take note we can only download stuff to the /uploads folder because it is owned by us.

&#x20;

There are a few interesting files within this.

&#x20;

!\[\[28\_Haircut\_image0010.png]]

I checked through all of them, and it looks like there is a certain exploit for the version of screen.

&#x20;

!\[\[28\_Haircut\_image0011.png]]

Read the exploit and follow it accordingly.

&#x20;

And this would pwn the box for us.

!\[\[28\_Haircut\_image0012.png]]

Get the flag and done.
