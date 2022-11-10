# Develop

Develop

Saturday, July 9, 2022

2:57 PM

Nmap scan:

{width="7.0in" height="0.78125in"}

&#x20;

!\[\[77\_Develop \_image002.png]]

&#x20;

Port 80:

!\[\[77\_Develop \_image003.png]]

&#x20;

Trying to get in touch results in this domain being revealed.

!\[\[77\_Develop \_image004.png]]

&#x20;

We can add this to our hosts file.

&#x20;

Bottom of the page reveals some form of users.

!\[\[77\_Develop \_image005.png]]

&#x20;

I find it very suspicious that there would be a getintouch.php, indicating to me that perhaps there is a user on the other end, or someone is clicking links or something.

&#x20;

&#x20;

Joinusnow reveals there is a CV Link there.

!\[\[77\_Develop \_image006.png]]

&#x20;

Perhaps this can be used for some phishing action.

&#x20;

Let's take a look at that git repository.

{width="6.895833333333333in" height="1.6875in"}

&#x20;

Straightaway, we find a username.

&#x20;

We can view more of the logs.

{width="8.385416666666666in" height="3.7291666666666665in"}

&#x20;

!\[\[77\_Develop \_image009.png]]

&#x20;

There's a staff vhost, and juicy files have been removed.

The passwords are still weak, and there is a commands thing?

&#x20;

We can delve deeper into the logs with -5 option.

&#x20;

!\[\[77\_Develop \_image0010.png]]

&#x20;

And we would have some usernames and passwords.

None of these hashes crack, which is really weird.

&#x20;

One thing I noticed is that the hash for one user starts with 0e, and the checking for the password does not use ===, only ==.

{width="6.28125in" height="2.5in"}

&#x20;

Perhaps magic hashes can be used here.

And this worked for the user lu191.

&#x20;

We can login with lu191:240610708.

&#x20;

At the dashboard, we can see this:

!\[\[77\_Develop \_image0012.png]]

&#x20;

There's a point of which we can execute pings, and this is clearly grounds for OSI.

&#x20;

Any form of special characters would be greeted with this.

!\[\[77\_Develop \_image0013.png]]

&#x20;

From more experimentation, it seems that this does not block out special characters, but rather the WAF detects the strings to see if there are commands within that.

!\[\[77\_Develop \_image0014.png]]

&#x20;

Stuff like xxd is allowed.

As it turns out, all we need to do is find the whitelisted commands, and be able to use ${IFS} to allow for successful execution of code.

!\[\[77\_Develop \_image0015.png]]

&#x20;

Cool.

With this, we can write a shell.

!\[\[77\_Develop \_image0016.png]]

&#x20;

Wget commands work, so all we need to do is put this file somewhere we can access.

&#x20;

So we have curl and wget.

We can actually leverage LFI in this manner.

!\[\[77\_Develop \_image0017.png]]

&#x20;

!\[\[77\_Develop \_image0018.png]]

This command is allowed.

&#x20;

So we can leverage this somehow to retrieve a user's SSH key, but first need to find out the username. Likely it would be franz. Then we need to figure out how to get it out.

!\[\[77\_Develop \_image0019.png]]

&#x20;

This file exists, indicated by the ping going through successfully.

&#x20;

Now, we need to think about how we can get this out.

&#x20;

!\[\[77\_Develop \_image0020.png]]

&#x20;

We can do something like this and have port 80 listening to read files.

!\[\[77\_Develop \_image0021.png]]

&#x20;

!\[\[77\_Develop \_image0022.png]]

&#x20;

Then we can do the same thing with the SSH key.

!\[\[77\_Develop \_image0023.png]]

&#x20;

Then we can SSH in as franz.

&#x20;

Flag:

!\[\[77\_Develop \_image0024.png]]

&#x20;

&#x20;

PE:

We can view the database.php file and see this.

!\[\[77\_Develop \_image0025.png]]

&#x20;

However, this cannot connect to mysql, unless we do something about it like port forwarding, which I'll try later.

When we see the user we are in, we can see this. (LinPEAS)

{width="10.03125in" height="2.0625in"}

&#x20;

Being a part of the docker group is great, meaning we can easily just mount onto the system as root.

WE can view the images present.

!\[\[77\_Develop \_image0027.png]]

&#x20;

Then we can just mount it with full privileges.

!\[\[77\_Develop \_image0028.png]]

&#x20;

Flag:

!\[\[77\_Develop \_image0029.png]]

&#x20;

If this is not counted as a shell:

!\[\[77\_Develop \_image0030.png]]

&#x20;

We can just create bash SUID.

&#x20;
