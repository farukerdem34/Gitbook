# Phobos

Phobos

Sunday, July 10, 2022

1:18 PM

Nmap scan:

!\[\[78\_Phobos \_image001.png]]

&#x20;

Only one port huh...

&#x20;

Port 80:

!\[\[78\_Phobos \_image002.png]]

&#x20;

We can gobust this to find some hidden stuff.

!\[\[78\_Phobos \_image003.png]]

&#x20;

There is this /internal directory that looks suspicious, but we aren't allowed to access any of the pages that are present there for whatever reason.

We can use feroxbuster's faster recursive function that would allow for us to view the data faster.

&#x20;

After loads of brute forcing using different wordlists, we can find a few good files.

!\[\[78\_Phobos \_image004.png]]

&#x20;

Looks like some kind of hidden repository, with the users folder being the endpoint. There are more endpoints that can be fuzzed through this.

&#x20;

When we try to access /svn, we can see that there is some form of authentication required.

!\[\[78\_Phobos \_image005.png]]

So we need to do something with the /internal, which looks a lot like there could be potential source code there. We should be finding credentials of some sort to access the hidden web server.

&#x20;

Gobusting /internal:

!\[\[78\_Phobos \_image006.png]]

&#x20;

We can attempt to brute force the authentication requirement using Hydra:

!\[\[78\_Phobos \_image007.png]]

&#x20;

Changed wordlists and didn't try the basic credentials...

!\[\[78\_Phobos \_image008.png]]

&#x20;

Next, we can use the credentials we found to access the hidden local repository.

&#x20;

Fuzzing /svn:

!\[\[78\_Phobos \_image009.png]]

&#x20;

!\[\[78\_Phobos \_image0010.png]]

&#x20;

!\[\[78\_Phobos \_image0011.png]]

&#x20;

Now, we can see all of this stuff. This is basically the repository for the hidden web server, of which case we can enumerate more of it

This application is basically a web shell for Django.

&#x20;

RCE identification:

!\[\[78\_Phobos \_image0012.png]]

&#x20;

Within the views.py, it seems that when we interact with this web app, we can see that there is a file upload, and that it is easy to inject OS commands into that using expressions.

&#x20;

We just need to figure out how to actually interact with the web application at this point.

&#x20;

Despite the looks of the website being a bit wrong, it still functions and is able to do stuff like delete and so on.

Now, we should be thinking of how to get the website to accept an upload.

&#x20;

We can see that a staff member is required for this, whatever that means, and then we can send POST requests to this web page, and from looking at it, there is an LFI there.

It basically reads any files, or executes commands.

&#x20;

In jobs.html, we can find this upload file thing.

!\[\[78\_Phobos \_image0013.png]]

&#x20;

But again, all of the links and functions to this website are broken and not usable, and it would appear we need to upload them ourselves.

&#x20;

There are remote management scripts for django admin interface, whereby it just sends requests and lets the thing process our requests.

Since we are unable to make any legitimate requests, we can just send the requests from our server to be processed by the remote Django API.

&#x20;

I have no idea what to do here, so I ran through the walkthrough to see what was it I had to do.

&#x20;

So from here, we can actually enumerate the SVN server using the svn command.

!\[\[78\_Phobos \_image0014.png]]

&#x20;

Then we can find the differences between each of the commits.

&#x20;

!\[\[78\_Phobos \_image0015.png]]

&#x20;

We can find a new virtual host.

Once we are there, we can go to the browser and see that it is actually the application's login page.

&#x20;

Based on the earlier directories, we can register a new user on this application.

!\[\[78\_Phobos \_image0016.png]]

&#x20;

!\[\[78\_Phobos \_image0017.png]]

&#x20;

From what we saw earlier, the submission had an RCE.

Let's check out the LFI.

&#x20;

However, we need to be a staff member or something. In this case, we can actually view the accounts page and attempt to reset the password.

!\[\[78\_Phobos \_image0018.png]]

&#x20;

!\[\[78\_Phobos \_image0019.png]]

&#x20;

Change the username to admin.

Then we can login to the server as the admin user.

!\[\[78\_Phobos \_image0020.png]]

&#x20;

LFI:

!\[\[78\_Phobos \_image0021.png]]

&#x20;

!\[\[78\_Phobos \_image0022.png]]

&#x20;

From here, we can remember that we saw some kind of ufw firewall.

!\[\[78\_Phobos \_image0023.png]]

&#x20;

!\[\[78\_Phobos \_image0024.png]]

&#x20;

We can use port 6000 for a reverse shell.

&#x20;

Shell:

!\[\[78\_Phobos \_image0025.png]]

&#x20;

{width="8.625in" height="2.2291666666666665in"}

&#x20;

PE:

We noticed earlier that there was some kind of database thingy.

&#x20;

We can enumerate the services running to find that mongodb is indeed on the server.

!\[\[78\_Phobos \_image0027.png]]

&#x20;

Download chisel to the server through port 6000 using wget.

!\[\[78\_Phobos \_image0028.png]]

&#x20;

!\[\[78\_Phobos \_image0029.png]]

&#x20;

Then we can enumerate the mongodb instance.

{width="3.78125in" height="0.8125in"}

&#x20;

!\[\[78\_Phobos \_image0031.png]]

&#x20;

!\[\[78\_Phobos \_image0032.png]]

&#x20;

We can crack this hash on crackstation.

!\[\[78\_Phobos \_image0033.png]]

&#x20;

We now have the root user's password.

&#x20;

And we can just su to become root.

!\[\[78\_Phobos \_image0034.png]]

&#x20;

Root Flag:

!\[\[78\_Phobos \_image0035.png]]

Local Flag:

!\[\[78\_Phobos \_image0036.png]]

&#x20;

&#x20;
