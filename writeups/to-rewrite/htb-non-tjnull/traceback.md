# Traceback

Traceback

Tuesday, 29 March 2022

4:04 pm

A Linux machine with these ports:

&#x20;

!\[\[14\_Traceback\_image001.png]]

When viewing the web port, this is what we get:

&#x20;

!\[\[14\_Traceback\_image002.png]]

&#x20;

!\[\[14\_Traceback\_image003.png]]

This is in the page source.

&#x20;

We can google this line, and we find [this Github Repo](https://github.com/TheBinitGhimire/Web-Shells).

&#x20;

After testing some of the names, we can determine that there is smevk.php.

&#x20;

!\[\[14\_Traceback\_image004.png]]

We can login with admin:admin, and from here, we can determine more web shells that are there.

&#x20;

{width="11.53125in" height="3.71875in"}

This already provides an RCE, so we can just connect back to our machine from this.

!\[\[14\_Traceback\_image006.png]]

&#x20;

!\[\[14\_Traceback\_image007.png]]

&#x20;

In our home directory, we can find this little file here.

!\[\[14\_Traceback\_image008.png]]

&#x20;

Lua is a programming language, so we need to find out where this file is.

!\[\[14\_Traceback\_image009.png]]

We can look at this, and we find that sysadmin can execute luvit.

&#x20;

!\[\[14\_Traceback\_image0010.png]]

&#x20;

From here, we can do this.

!\[\[14\_Traceback\_image0011.png]]

&#x20;

Now we are sysadmin.

Grab the user flag and continue on.

Ran a linpeas here and wanted to see what groups we could access.

!\[\[14\_Traceback\_image0012.png]]

Since we can write to this, we can just make it such that we execute a script on SSH login. Echo our public key into the authorized\_keys of sysadmin and then do this command.

!\[\[14\_Traceback\_image0013.png]]

&#x20;

When we SSH in, it would create the bash file there for us to execute to become root.

!\[\[14\_Traceback\_image0014.png]]

Pwned.

&#x20;

&#x20;
