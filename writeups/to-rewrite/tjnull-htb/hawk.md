# Hawk

Hawk

Sunday, 6 March 2022

2:00 pm

This is a Linux machine.

The target IP is 10.10.10.102.

My IP is 10.10.16.2.

&#x20;

Enumeration times.

!\[\[69\_Hawk\_image001.png]]

Let's do a more detailed scan, in the meantime check out that ftp server too.

&#x20;

Detailed scan:

!\[\[69\_Hawk\_image002.png]]

Not too sure what these ports are, and that one port 5435 seems to be blocked by a firewall.

Port 9092 is some kind of XML server or something.

&#x20;

FTP:

This allows for anonymous logins.

!\[\[69\_Hawk\_image003.png]]

!\[\[69\_Hawk\_image004.png]]

&#x20;

One directory is present, so let's take a look inside of it.

!\[\[69\_Hawk\_image005.png]]

&#x20;

!\[\[69\_Hawk\_image006.png]]

There's an encrypted file, so let's take it for now and prepare to use openssl to decrypt it or something. Let's see what we can get without a passphrase.

!\[\[69\_Hawk\_image007.png]]

This gave me a base64 string.

&#x20;

!\[\[69\_Hawk\_image008.png]]

&#x20;

!\[\[69\_Hawk\_image009.png]]

Intriguing. Salted\_\_kY? Perhaps I need a passphrase or something to decrypt this file further.

&#x20;

Let's take a look at the web servers.

&#x20;

!\[\[69\_Hawk\_image0010.png]]

This seems to be running on Drupal, and this is all there is.

We seem to be able to create an account. However, when tried, there is an error message. Turns out we need a proper email address in order to bypass this validation.

!\[\[69\_Hawk\_image0011.png]]

Ran a directory enumeration on this one, to see what we get.

We could run drupalgeddon on this, but it does not work well.

&#x20;

I wanted to look at the encrypted file again and attempt to brute force this one, because everything else is not getting me anywhere.

Downloaded go-openssl-bruteforce to do some brute forcing for me.

&#x20;

{width="6.59375in" height="4.208333333333333in"}

Right, so let's get into that box.

&#x20;

Logged in as admin.

!\[\[69\_Hawk\_image0013.png]]

From here, let's look around and see what we can do on this web engine.

&#x20;

Noticed that we can add content in the form of PHP files.

!\[\[69\_Hawk\_image0014.png]]

Let's try injecting some PHP code to execute bash to give us a shell.

&#x20;

!\[\[69\_Hawk\_image0015.png]]

Didn't work, so let's take a look at the configurations.

&#x20;

There's this one checkbox we can tick.

!\[\[69\_Hawk\_image0016.png]]

Let's allow this and try again.

&#x20;

!\[\[69\_Hawk\_image0017.png]]

Select PHP code.

&#x20;

When uploaded and executed, it will give us a shell.

!\[\[69\_Hawk\_image0018.png]]

We are www-data.

&#x20;

Better the shell:

{width="4.4375in" height="2.3229166666666665in"}

&#x20;

We have access to the user daniel and can grab the user flag while we're in here.

Looked around in the html file and found this:

!\[\[69\_Hawk\_image0020.png]]

This was in the /var/www/html/sites/default/settings.php file.

Tried an SSH and interestingly, this changes us to use a python shell.

&#x20;

!\[\[69\_Hawk\_image0021.png]]

Just do this command to get us a shell:

{width="7.427083333333333in" height="1.40625in"}

Now let's take a look around properly.

&#x20;

Interestingly, there's this one h2 directory.

!\[\[69\_Hawk\_image0023.png]]

&#x20;

When viewing the open ports, we can see the port 8082 is still running and it is a HTTP port.

!\[\[69\_Hawk\_image0024.png]]

Let's try some port relaying to see what we can get from this one web server that initially rejected our connection request.

&#x20;

!\[\[69\_Hawk\_image0025.png]]

Now access the web server.

&#x20;

!\[\[69\_Hawk\_image0026.png]]

Now, from hacktricks (which revealed the box answer...) we can simply change the /test to /\<anything we want> and then we can log in.

&#x20;

A quick google search led me to H2 SQL Command Chaining.

By inputting some commands, we are able to execute arbitrary code.

&#x20;

!\[\[69\_Hawk\_image0027.png]]

&#x20;

Let's try gaining a reverse shell.

Tried for really long, but was unable to gain any reverse shells...So let's try some other methods.

&#x20;

!\[\[69\_Hawk\_image0028.png]]

&#x20;

!\[\[69\_Hawk\_image0029.png]]

For some reason, I was able to get this instead of reverse shells. Something must be blocking reverse shells.

&#x20;

Perhaps I could add daniel as an admin. I kept trying other commands and stuff, bnu

Think I crashed the machine while doing this method. So I had to reset :C.

&#x20;

Anyways I realised I can grab the root flag from this, but that's not why we're here.

&#x20;

!\[\[69\_Hawk\_image0030.png]]

Tried this script from our searchsploit. This was in line with the H2 Database server version as well.

&#x20;

Uploaded it onto daniel's account and ran it.

{width="5.375in" height="2.375in"}

There we go.

&#x20;

&#x20;
