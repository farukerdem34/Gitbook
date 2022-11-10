# Tabby

Tabby

Tuesday, 15 February 2022

6:05 pm

This is a Linux machine.

My IP is 10.10.16.9.

The target IP is 10.10.10.194,

&#x20;

Nmap scan. Fast scan reveals there to be SSH, HTTP and port 8080 for HTTP open on the machine.

&#x20;

!\[\[41\_Tabby\_image001.png]]

I see tomcat, and I know that this is vulnerable.

&#x20;

Anyways I ran directory scan on both websites.

!\[\[41\_Tabby\_image002.png]]

Took a long break for this one, now back to it.

Taking a closer look at the webpage, we can see that there is this megahosting.htb.

&#x20;

!\[\[41\_Tabby\_image003.png]]

This could perhaps be a hint towards another domain that is on the server.

&#x20;

There's this link to view some form of data breach, and when clicking it, it brings us to this particular page.

!\[\[41\_Tabby\_image004.png]]

&#x20;

I found the specification of a filename quite...suspicious. No other part of the website includes some form of file name. I played with this parameter, and SQL injection was my first guess. However, this yielded nothing, SQLmap ruled that this was non-injectable.

&#x20;

I tried to view the /etc/passwd file through directory traversal as well. This yielded good results! (Thank you Portswigger Academy, with all your filename fuzzing)

&#x20;

!\[\[41\_Tabby\_image005.png]]

&#x20;

So we have a directory traversal that works and we can get an LFI...now what?

&#x20;

Clearly we need to use this to execute reverse shells or read some form of credentials.

Ok, so some Googling led me to believe that Tomcat actually stores the passwords within the folder on the machine. I think I may have to use this file in order to get to the password files.

&#x20;

I know that the web browser is somewhere in /var/www, so let's start there.

Looking at the documentation of the tomcat manager application, it tells us this file.

!\[\[41\_Tabby\_image006.png]]

&#x20;

Btw, the default credentials do not work.

I just installed tomcat9, and looked at the directory on my own machine.

&#x20;

!\[\[41\_Tabby\_image007.png]]

The second one works, and I tried it in burpsuite since the browser was just not cooperative with me.

&#x20;

!\[\[41\_Tabby\_image008.png]]

&#x20;

!\[\[41\_Tabby\_image009.png]]

&#x20;

There we go, now let's log in.

!\[\[41\_Tabby\_image0010.png]]

&#x20;

Cool.

Now Tomcat allows us to upload .war files, which we can generate using MSFVenom.

!\[\[41\_Tabby\_image0011.png]]

This should work, and according to hacktricks, we can just upload this using curl.

!\[\[41\_Tabby\_image0012.png]]

&#x20;

To activate this, just load it!

!\[\[41\_Tabby\_image0013.png]]

We definitely need to upgrade this shell somehow.

!\[\[41\_Tabby\_image0014.png]]

&#x20;

Better. Anyways enumeration tells me there's a user called ash on the machine.

!\[\[41\_Tabby\_image0015.png]]

Access is denied however.

I wanted to take a look at the news.php file, which is what caused the LFI.

{width="6.083333333333333in" height="5.15625in"}

And I found something owned by the user as well.

&#x20;

There seems to be a backup zip...hmm...

!\[\[41\_Tabby\_image0017.png]]

&#x20;

I wonder if..this is the thing we need to get his password and SSH in? Perhaps.

&#x20;

Anyways this zip is password protected, so we definitely need john or something to crack it.

!\[\[41\_Tabby\_image0018.png]]

Googled around and found a way to transfer it to my computer easily.

!\[\[41\_Tabby\_image0019.png]]

&#x20;

!\[\[41\_Tabby\_image0020.png]]

&#x20;

Now, I think we can zip2john this thing.

!\[\[41\_Tabby\_image0021.png]]

&#x20;

{width="10.322916666666666in" height="2.2604166666666665in"}

Great, now let's unzip this thing.

Unfortunately, there's nothing in this zip file. However, we did have a new password. And why not try it?

Worked!

!\[\[41\_Tabby\_image0023.png]]

Always check every single password.

We can't run sudo on this computer as ash.

&#x20;

So we need to look to other things that we can do.

I downloaded over the LinEnum.sh file, and this caught my eye.

{width="9.447916666666666in" height="0.6666666666666666in"}

A little bit of Googling tells me that with lxd privileges, we can actually become root instantly. Lxd is indeed installed on the machine as well.

!\[\[41\_Tabby\_image0025.png]]

Followed hacktricks to do this, no way I would have done it myself.

[https://steflan-security.com/linux-privilege-escalation-exploiting-the-lxc-lxd-groups/](https://steflan-security.com/linux-privilege-escalation-exploiting-the-lxc-lxd-groups/)

This page is great for that.

&#x20;

Anyways, here is the result of the exploit.

!\[\[41\_Tabby\_image0026.png]]

&#x20;

!\[\[41\_Tabby\_image0027.png]]

Pwned. Cool box! Definitely being one of the more fun ones around. LXC exploitation and creating my own container of which we can be the root user is really cool.
