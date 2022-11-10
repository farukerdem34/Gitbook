# Hunit

Hunit

Tuesday, 22 March 2022

8:05 pm

A Linux machine with these ports:

!\[\[31\_Hunit \_image001.png]]

&#x20;

!\[\[31\_Hunit \_image002.png]]

&#x20;

Interestingly, when visiting port 8080, there are some Haikus available for us to read.

&#x20;

!\[\[31\_Hunit \_image003.png]]

&#x20;

When viewing one of the haikus and reading the page source, we can find this little link here.

!\[\[31\_Hunit \_image004.png]]

&#x20;

There's an /api directory which indicates we need to do some API pentesting regarding this application.

!\[\[31\_Hunit \_image005.png]]

&#x20;

Interesting.

So we can query about users and stuff as well.

!\[\[31\_Hunit \_image006.png]]

&#x20;

We receive some usernames and passwords. We can test this with SSH.

&#x20;

The creds for dademola are the correct ones (duh, we have a clear password there).

{width="9.822916666666666in" height="2.53125in"}

&#x20;

Flag:

!\[\[31\_Hunit \_image008.png]]

F592970260dd5a0c25d3eea255428f2c

&#x20;

PE:

Within the /home directory, there's another user here.

{width="6.333333333333333in" height="1.4375in"}

&#x20;

There's a git user.

And there are also SSH keys within this user's home directory.

{width="6.8125in" height="2.1354166666666665in"}

&#x20;

With this, we can SSH by taking the private key from this user.

{width="8.166666666666666in" height="1.2395833333333333in"}

&#x20;

!\[\[31\_Hunit \_image0012.png]]

&#x20;

{width="5.09375in" height="1.375in"}

&#x20;

Now, this is clearly a git shell, which relies on commands being placed into the git-shell-commands folder located within the user's file.

&#x20;

Interestingly, within the main directory, there's this git-server here.

!\[\[31\_Hunit \_image0014.png]]

&#x20;

&#x20;

When viewing the crontabs, we can see this.

{width="5.125in" height="0.6979166666666666in"}

&#x20;

So there's a backups.sh being made consistently, and there's a pull being made. This would indicate that perhaps there are consistent pull requests being made by the root user.

&#x20;

Perhaps there is a way to escape this restricted shell and access git with a shell.

Googling sdtuff, we can see how there are ways to spawn a shell using some commands.

There are exploits to escape this:

[https://insinuator.net/2017/05/git-shell-bypass-by-abusing-less-cve-2017-8386/](https://insinuator.net/2017/05/git-shell-bypass-by-abusing-less-cve-2017-8386/)

&#x20;

Through this, we can execute git shell commands.

{width="9.697916666666666in" height="1.21875in"}

&#x20;

This has shown to be successful. This works by using -t to spawn a custom TTY to execute commands. Now the question is, we should be able to get this repository and make changes to it.

&#x20;

The git-shell as git should have some privileges over this folder.

&#x20;

!\[\[31\_Hunit \_image0017.png]]

&#x20;

These are the commands that can be done using this.

&#x20;

We can test this:

!\[\[31\_Hunit \_image0018.png]]

&#x20;

So here's our attack plan:

* Clone the repository and edit the backup.sh to have malicious commands
* Use the git-shell to push the changes to the main branch
* Wait for the root crontab to execute the backup.sh with our commands

&#x20;

Let's start by cloning it and adding a line to the backups.sh.

{width="7.75in" height="1.4583333333333333in"}

&#x20;

Now, let's try to see if we can push changes to the master branch of this repo.

&#x20;

{width="6.739583333333333in" height="0.7916666666666666in"}

Now we have inserted the file, and we just need to push this back to the master branch.

We can try to push as the user, but it wouldn't work.

{width="7.75in" height="2.4166666666666665in"}

&#x20;

So in this case, we can try to download the repository using the git-shell.

!\[\[31\_Hunit \_image0022.png]]

&#x20;

Then, we can add our own files into the repository after defining our config stuff.

!\[\[31\_Hunit \_image0023.png]]

&#x20;

!\[\[31\_Hunit \_image0024.png]]

&#x20;

Then we need to set up a listener port and push the changes back to the master branch.

!\[\[31\_Hunit \_image0025.png]]

&#x20;

Then we just have to wait for a shell to pop.

For some instances, we have to play around a bit with the ports chosen. But eventually it works. I found that using different ports for HTTP server and testing them with wget using the SSH access was the most reliable way to test for good ports.

&#x20;

!\[\[31\_Hunit \_image0026.png]]

&#x20;

Flag:

!\[\[31\_Hunit \_image0027.png]]

&#x20;

&#x20;

&#x20;
