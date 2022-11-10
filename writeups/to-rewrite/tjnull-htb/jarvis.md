# Jarvis

Jarvis

Sunday, 6 February 2022

8:03 pm

This is a Linux machine.

The target IP is 10.10.10.143

My IP is 10.10.16.5.

&#x20;

Preliminary scans show that port 22 and port 80 is open.

&#x20;

!\[\[20\_Jarvis\_image001.png]]

&#x20;

Knowing that there's a port 80 open, I ran directory enumeration as well.

&#x20;

This box has to do with SQL injections, so let's take a look at the web page.

!\[\[20\_Jarvis\_image002.png]]

&#x20;

Take note of the domain name in case as well. Nothing of interest there.

!\[\[20\_Jarvis\_image003.png]]

&#x20;

There's a phpmyadmin page where we can log in.

!\[\[20\_Jarvis\_image004.png]]

&#x20;

Perhaps this could be exploited.

&#x20;

When we enter a wrong password, it seems that we are denied access to the website's MySQL server.

!\[\[20\_Jarvis\_image005.png]]

&#x20;

I would brute force stuff, but it seems that there is a measure to counter that.

&#x20;

!\[\[20\_Jarvis\_image006.png]]

&#x20;

We would need to enumerate other ways to break in.

&#x20;

Anyways I did another directory enumeration on this phpmyadmin site, and this returned a lot of useful directories.

!\[\[20\_Jarvis\_image007.png]]

&#x20;

Checking out the /doc/html/user.html reveals that this is phpMyAdmin 4.8.0. Nice!

!\[\[20\_Jarvis\_image008.png]]

/setup shows me some form of panel of which I can change configurations and what not.

&#x20;

!\[\[20\_Jarvis\_image009.png]]

&#x20;

We'll just take note of this for now.

&#x20;

There is an interesting file in /examples/signon.php.

&#x20;

!\[\[20\_Jarvis\_image0010.png]]

&#x20;

Not too sure what to make out of this.

&#x20;

Anyways, I didn't find anything new. I went back to enumerating the first page more. Perhaps I missed something.

&#x20;

I took note of this room.php thing. It seems that out of all the pages on the website, there is only one with a query parameter **cod**.

&#x20;

Suspicious!

!\[\[20\_Jarvis\_image0011.png]]

&#x20;

I managed to break the whole website by using that cod parameter and replacing it with **'**

&#x20;

Perhaps it's time to SQLMap this just to check.

Reports that this paramter is indeed vulnerable to UNION injection.

!\[\[20\_Jarvis\_image0012.png]]

&#x20;

This just outputted the database and stuff. Checked what passwords are present on the webserver and found this.

&#x20;

{width="11.135416666666666in" height="4.270833333333333in"}

&#x20;

Jackpot. Dbadmin:imissyou

&#x20;

From here we can log in to the phpMyAdmin page.

&#x20;

!\[\[20\_Jarvis\_image0014.png]]

&#x20;

From here, we just need to find an RCE outlet of some sort.

!\[\[20\_Jarvis\_image0015.png]]

&#x20;

Used the RCE one, and this works using directory traversal and stuff. This works out well!

{width="7.708333333333333in" height="2.5625in"}

&#x20;

Now just to slap on a reverse shell and get a listener port and we're good.

&#x20;

!\[\[20\_Jarvis\_image0017.png]]

&#x20;

!\[\[20\_Jarvis\_image0018.png]]

&#x20;

Make the shell stable and we're able to get in.

!\[\[20\_Jarvis\_image0019.png]]

&#x20;

From here we need to priv escalate.

&#x20;

I did a sudo -l and it seems that the file owned by the user, **pepper**, is able to run this one file in python.

!\[\[20\_Jarvis\_image0020.png]]

&#x20;

See if we can echo some stuff in to make it such that we control the shell.

&#x20;

Analysis of the code within this file reveals that it is able to ping users? Uncommon for python scripts really.

!\[\[20\_Jarvis\_image0021.png]]

&#x20;

Further analysis of the code within the script shows the exec\_ping() function, which directly takes the IP given and pings it. There's no user input validation besides for a few characters that cannot be entered at all costs. Interesting, meaning it just calls the system to ping instead. This could be a potential LFI or RCE, and we will remember this for now.

&#x20;

!\[\[20\_Jarvis\_image0022.png]]

The interesting thing is that it does not cover **$**. This would mean that bash scripts are able to be used in some way. I just know that without the $, we are able to do something. It's clearly a weakness. I tried just inputting $(/bin/bash) for fun, because I at least know that the $ sign has to be followed by brackets.

&#x20;

Surprisingly, this gave me a sort of shell.

&#x20;

!\[\[20\_Jarvis\_image0023.png]]

&#x20;

I admit I had no idea what I was doing. I can't get any output from this shell, so this is not the answer. The next step would be to think of how I can execute some sort of file.

Perhaps in the file I can write some reverse shell and get it back to my machine.

I remembered that I can create some .sh files and execute that.

&#x20;

Did the following.

!\[\[20\_Jarvis\_image0024.png]]

&#x20;

I did so as pepper, because while I could not get any output, I was sure that I could change directories and execute commands. This would allow me to make the file executable.

&#x20;

From here, I executed the file using the same method I did to execute /bin/bash.

&#x20;

This gave me a user shell.

!\[\[20\_Jarvis\_image0025.png]]

&#x20;

!\[\[20\_Jarvis\_image0026.png]]

&#x20;

From here we need to priv escalate to the next level. I had LinEnum.sh prepared!

!\[\[20\_Jarvis\_image0027.png]]

&#x20;

{width="7.75in" height="1.3020833333333333in"}

This interesting file popped up. SystemCTL can be indeed used to privilege escalate. [This](https://gist.github.com/A1vinSmith/78786df7899a840ec43c5ddecb6a4740) is a good resource we can use.

A writable source is /dev/shm, which we can use .

&#x20;

This is our payload.

{width="7.302083333333333in" height="5.28125in"}

&#x20;

Run it and we gain the root shell.

&#x20;

!\[\[20\_Jarvis\_image0030.png]]

&#x20;

!\[\[20\_Jarvis\_image0031.png]]

&#x20;

This was a fun box, I managed to use LinEnum successfully and do my own research regarding the usage of text editors within the CLI.

1. Learnt that cat > filename <\<EOF is a fantastic way of writing my own shit.
2. Accidentally found the /bin/bash exploit, learnt that $ signs are used as an expression of bash, that's why shells start with $ or #, depending in the level of access the user is at.

&#x20;

Overall, a good box. 5/5.

&#x20;

&#x20;
