# Maria

Maria

Tuesday, July 5, 2022

12:15 PM

Nmap scan:

!\[\[72\_Maria \_image001.png]]

&#x20;

&#x20;

Port 80:

!\[\[72\_Maria \_image002.png]]

&#x20;

Wordpress is good to start with, and we can begin with wpscan.

&#x20;

!\[\[72\_Maria \_image003.png]]

&#x20;

There are some vulnerabilities.

&#x20;

FTP:

Anonymous access is allowed, but there is nothing much on it besides loads of hints about the usage of automysqlbackup.

&#x20;

I don't have much choice here, so I started brute forcing the login as the administrator and also figuring out what to do with this mysql backup stuff.

&#x20;

Right, so we can notice the type of file structure here.

&#x20;

Let's re-enumerate the webapp, there has to be something.

Wpscan is obviously not picking up on some stuff.

&#x20;

!\[\[72\_Maria \_image004.png]]

&#x20;

Akismet and duplicator 1.3.26.

Akismet has some stored XSS exploits while duplicator has directory traversal exploits.

&#x20;

[https://www.exploit-db.com/exploits/50420](https://www.exploit-db.com/exploits/50420)

This one works.

!\[\[72\_Maria \_image005.png]]

&#x20;

!\[\[72\_Maria \_image006.png]]

&#x20;

Joseph is the user here.

&#x20;

!\[\[72\_Maria \_image007.png]]

&#x20;

There are files within this automysqlbackup, and this has etc, usr and var...?

!\[\[72\_Maria \_image008.png]]

&#x20;

Within the /usr/sbin file, there is this autosqlbackup.

We can read this.

!\[\[72\_Maria \_image009.png]]

&#x20;

We can then find the conf file.

!\[\[72\_Maria \_image0010.png]]

&#x20;

We can enumerate the FTP a bit more to find another file at /etc/default/automysqlbackup.

!\[\[72\_Maria \_image0011.png]]

&#x20;

With these credentials, we can log in.

{width="8.354166666666666in" height="2.6666666666666665in"}

&#x20;

!\[\[72\_Maria \_image0013.png]]

&#x20;

Then we can crack this hash. I don't think it's healthy for my processor to be working this long.

But this hash does not crack with rockyou.txt

&#x20;

Interesting.

Take note we did find another form of XSS there, and perhaps we would have to exploit the MySQL database through uploading files instead.

!\[\[72\_Maria \_image0014.png]]

&#x20;

I tried to brute force this thing using online sources.

Didn't work.

&#x20;

So the hash doesn't crack.

Here are our options:

* Find a way to reset the wordpress password (I have no idea how)
* Find a way to inject the payload within the database.
* Find a way to dump the entire database and painstakingly go through the data to find what we need.
* Use Akismet XSS to do stuff with the token.

&#x20;

All of which are honestly really far-fetched but not impossible.

&#x20;

!\[\[72\_Maria \_image0015.png]]

&#x20;

After a bit of research, it appears that it is possible to theoretically find the debug log of which the email is located within the wordpress site and be able to intercept the email after pressing forgot password, and we can then reset the password of the administrator.

!\[\[72\_Maria \_image0016.png]]

&#x20;

The SMTP plugin has debug on.

&#x20;

!\[\[72\_Maria \_image0017.png]]

&#x20;

And the plugin, along with duplicator, is on, meaning that technically, with the LFI, there is a way of which we can view these files. We just need to find out how to view it, and hopefully be able to reset the administrator password.

&#x20;

When we reset the password, we can view the wp\_users again.

!\[\[72\_Maria \_image0018.png]]

&#x20;

We get this, which is an activation key that is used for the password reset.

This is not possible to exploit however, as there are no ways for us to decrypt this randomly generated string.

&#x20;

Options so far:

* Intercepting mail?
  * Impossible because well the name is randomly generated and cannot be intercepted.
* Cracking hash
  * Already tried with the entire rockyou.txt, we can try again but it's pointless.
* Modify database in any way?
  * User does not have permissions
* Other vulnerabilities?
  * Then what's the point of using MySQL database.
* Resetting password?
  * The hash for the key is a random thing
  * Could be within the database somewhere.

&#x20;

Let's look at the database for that log file and then we can proceed.

The version of smtp here is vulnerable because the log file is open to public.

&#x20;

!\[\[72\_Maria \_image0019.png]]

&#x20;

The log file is there.

!\[\[72\_Maria \_image0020.png]]

&#x20;

Then we can find this and reset the password of the administrator.

!\[\[72\_Maria \_image0021.png]]

&#x20;

Then we can log in, and now we can do the usual wordpress shell.

Go to Theme Editor and replace the 404.php with a shell.

&#x20;

{width="4.583333333333333in" height="1.625in"}

!\[\[72\_Maria \_image0023.png]]

&#x20;

!\[\[72\_Maria \_image0024.png]]

&#x20;

Then we can get a shell.

&#x20;

!\[\[72\_Maria \_image0025.png]]

&#x20;

{width="9.010416666666666in" height="1.7083333333333333in"}

&#x20;

Flag:

!\[\[72\_Maria \_image0027.png]]

bcfa9ba877e12b57a68183a5c6a5f88b

&#x20;

PE:

We can run LinPEAS and pspy for some enumeration.

&#x20;

!\[\[72\_Maria \_image0028.png]]

&#x20;

There are quite a few processes running here.

Exim4 here is vulnerable to a heap based overflow, meaning that we could potentially use this to gain a shell.

This is because exim4 is also an SUID binary.

&#x20;

There are vulnerabilities for this.

&#x20;

Particularly, there is one Heap Based Overflow that exists that supposedly allows for RCE, but I have been unable to get it to work.

&#x20;

Automysqlbackup:

When executing this one, w can see that it appears to look for scripts within a directory that we can control.

!\[\[72\_Maria \_image0029.png]]

&#x20;

When looking at the source code, we can see that it simply executes whatever script that we put inside the directory.

&#x20;

{width="8.854166666666666in" height="2.6354166666666665in"}

&#x20;

So all we need to do is put a script there. And we should name it accordingly.

!\[\[72\_Maria \_image0031.png]]

&#x20;

!\[\[72\_Maria \_image0032.png]]

&#x20;

Then we just wait and we would get a root shell.

{width="8.875in" height="2.2291666666666665in"}

&#x20;

Flag:

!\[\[72\_Maria \_image0034.png]]

&#x20;

Always look at funny binaries and don't get caught up in weird rabbit exim holes.

&#x20;
