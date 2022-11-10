# Breakout

Breakout

Wednesday, July 6, 2022

12:46 PM

Nmap scan:

!\[\[73\_Breakout\_image001.png]]

&#x20;

&#x20;

Port 80:

!\[\[73\_Breakout\_image002.png]]

&#x20;

We can register.

!\[\[73\_Breakout\_image003.png]]

&#x20;

!\[\[73\_Breakout\_image004.png]]

&#x20;

This does not work.

&#x20;

We can also find a possible version through the usage of the /help windows.

!\[\[73\_Breakout\_image005.png]]

&#x20;

Seems that we are running 13.9.

&#x20;

Through some fuzzing, we can find a few directories.

!\[\[73\_Breakout\_image006.png]]

&#x20;

In similar fashion, we can find the administrator user as well.

!\[\[73\_Breakout\_image007.png]]

&#x20;

Who happens to be named root.

&#x20;

We can also find our own user.

!\[\[73\_Breakout\_image008.png]]

Who is blocked.

&#x20;

So at least we have some usernames, so we should be looking for more usernames, since their public repos are viewable by us.

&#x20;

GraphQL:

After a lot of looking around at other endpoitns found, I found a graphql endpoint.

&#x20;

!\[\[73\_Breakout\_image009.png]]

&#x20;

!\[\[73\_Breakout\_image0010.png]]

&#x20;

&#x20;

From here, we can find two other users, called michelle and coaran.

&#x20;

We can sign in with michelle michelle now.

&#x20;

Then, we can view the version of which is being used.

!\[\[73\_Breakout\_image0011.png]]

&#x20;

Now that we have access to this, perhaps we can experiement with the following options:

* RCE through creating packages.
* SSRF through importing packages
* Image RCE.

&#x20;

The first one does not work because we are unable to create any projects due to the fact that we don't have any SSH keys that are imported into the server, and hence we cannot authenticate as the user.

&#x20;

!\[\[73\_Breakout\_image0012.png]]

&#x20;

This would very likely be vulnerable to the image RCE, as the patches made are only for 13.9.4 and not this one.

[https://github.com/CsEnox/Gitlab-Exiftool-RCE](https://github.com/CsEnox/Gitlab-Exiftool-RCE)

&#x20;

It is also named after the creator, which is cool.

&#x20;

!\[\[73\_Breakout\_image0013.png]]

&#x20;

{width="8.447916666666666in" height="2.21875in"}

&#x20;

We would need to fix the path of this user.

!\[\[73\_Breakout\_image0015.png]]

&#x20;

What's weird is that the home is there, indicating to be that perhaps, this may be some kind of container.

&#x20;

PE:

LinPEAS output:

!\[\[73\_Breakout\_image0016.png]]

&#x20;

We are indeed within this container.

Breaking out is the first thing.

&#x20;

Interesting files:

{width="5.71875in" height="2.15625in"}

&#x20;

{width="7.28125in" height="1.5in"}

Hashes of users are found.

&#x20;

We can run john to crack these and continue enumeration.

&#x20;

There are indeed SSH keys found here.

!\[\[73\_Breakout\_image0019.png]]

&#x20;

With this key, we can SSH in as coaron.

&#x20;

Flag:

!\[\[73\_Breakout\_image0020.png]]

&#x20;

PE2:

I ran linPEAS and pspy on two different shells to look at what's going on.

&#x20;

!\[\[73\_Breakout\_image0021.png]]

&#x20;

There was this output here, and there is a backup.sh script located there. This is part of the cronjobs.

&#x20;

!\[\[73\_Breakout\_image0022.png]]

&#x20;

This has a wilcard, and it quite obviously easily exploited.

&#x20;

So let's make a quick script for the files and then just try to put it somewhere within that file.

&#x20;

We cannot edit the log files in any way shape or form, so there are a few options:

* Use the docker container to create our script
* Use the gitlab instance to create the script.
* Find a way to edit the files to contain our script.
* Find a local privilege vulnerability with these packages

&#x20;

The container has little privileges, and it probably cannot be used to create new logs or something. This could be used to create our malicious files, and be able to execute a command or two.

&#x20;

When we head to the /var/log file on the container, we would see the exact same files. And we can edit some of them.

!\[\[73\_Breakout\_image0023.png]]

&#x20;

We can make copies and see if the docker is the one exporting these log files.

!\[\[73\_Breakout\_image0024.png]]

&#x20;

We can see that our test files have made it to the machine itself. We can actually use this to create symlinks that are editable or something.

Alternatively, we can place an SUID binary or something here. Then we can basically write files to the main system and cause it to be archived somewhere.

&#x20;

The symlink method can be used first to archive the root user's SSH key, but we just need the git shell to do that.

&#x20;

!\[\[73\_Breakout\_image0025.png]]

&#x20;

We just do this, and then let the script execute it by waiting for a few minutes.

&#x20;

!\[\[73\_Breakout\_image0026.png]]

&#x20;

Then we can SSH in as root.

Flag:

!\[\[73\_Breakout\_image0027.png]]

2369ce74b4acf30e2dc10211116fbfb5\


&#x20;

&#x20;

&#x20;

&#x20;
