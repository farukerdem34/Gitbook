# Valentine

Valentine

Tuesday, 1 February 2022

11:25 pm

This is a Linux box.

My IP is 10.10.17.233.

Target IP is 10.10.10.79.

&#x20;

Enumeration:

&#x20;

!\[\[10\_Valentine\_image001.png]]

&#x20;

Second Nmap scan reveals that this is on port 80 and port 443, running an Apache server, so take note that directory and Vhost can be exploited.

&#x20;

!\[\[10\_Valentine\_image002.png]]

&#x20;

Quick dirbuster reveals that there are a few directories within the page, all of which are interesting. Also to take note that there are no apache httpd 2.2.22 exploits on searchsploit, nothing of interest to me.

&#x20;

!\[\[10\_Valentine\_image003.png]]

&#x20;

One by one, starting with /dev. This is a web directory that contains two files, 'hype\_key' and 'notes.txt'

&#x20;

Within /dev/notes.txt, there is a to do list, and it talks about the decode and encode directories present within the website.

Interesting!

&#x20;

!\[\[10\_Valentine\_image004.png]]

Hype key is a bunch of hex code.

&#x20;

!\[\[10\_Valentine\_image005.png]]

&#x20;

Moving onto /omg and /index, there is nothing of interest in either, which just displays the web page and the picture they have uploaded to be the homepage.

&#x20;

When visiting the /encode and /decode directories, it seems that this uses an encryption oracle of sorts. Whatever is encoded within /encode can be decoded using /decode. From notes.txt, we can see that all of it is done client-side, hence there is no processing within the server.

&#x20;

Decoding the hex reveals that this is indeed a private key! I have saved this as privkey.txt within my machine.

!\[\[10\_Valentine\_image006.png]]

&#x20;

Understanding that there is an SSH port open, and we have a private key, my next guess was whether we are allowed to SSH using this private key. This seems plausible, as the private key is the one used for authentication and stuff. The real answer lies in decrypting that private key, and we would be able to get a password from it! However, that requires a passphrase to be used to decrypt this key...

&#x20;

I ran a quick nmap vuln script scan, but and that returned **Heartbleed** vulnerability, which has vulnerabilities on exploitdb!

!\[\[10\_Valentine\_image007.png]]

&#x20;

!\[\[10\_Valentine\_image008.png]]

&#x20;

Heartbleed seems to be a memory disclosure kind of exploit, which would leak further information about the website. I ran a quick script using 32764.py and it revealed a bunch of data. There's this little snippet here, which seems to be base64.

!\[\[10\_Valentine\_image009.png]]

&#x20;

Decoding this gives us **heartbleedbelievethehype**. This seems to be the passphrase we are looking for!

&#x20;

Anyways back to openssl and decrypting that RSA certification. It worked out well using this passphrase.

!\[\[10\_Valentine\_image0010.png]]

&#x20;

From there, I figured out with some Google-fu that we have to change the extension to .key and then it works out well, and also we need to use the -i flag within SSH as an identity file of which we use to log in. We are definitely not logging in as root, hence I believe that we are logging in as user 'hype'. Just like that, we're in.

&#x20;

!\[\[10\_Valentine\_image0011.png]]

Grab the user flag and time to privilege escalate!

&#x20;

A quick find search reveals this to us. We have some privileges here and there, typically in directory /at.

&#x20;

!\[\[10\_Valentine\_image0012.png]]

&#x20;

Unfortunately, this did not work out in the end. I then listed all the processes that are run as root! All of them are not interesting, except for one.

&#x20;

{width="9.260416666666666in" height="1.6666666666666667in"}

&#x20;

!\[\[10\_Valentine\_image0014.png]]

&#x20;

Tmux seems to be running, and a bit of google fu reveals that the command '**tmux -S /.devs/dev\_sess'** would attach us to the root user. Running it gets us the flag! This is because tmux basically opens another terminal screen that is not on the main screen, hence by attaching ourselves to the tmux screen that is running as root, we can basically just get ourselves to that shell.

&#x20;

&#x20;

!\[\[10\_Valentine\_image0015.png]]

&#x20;

Pwned! Fun box!

What I learnt:

* Heartbleed vulnerability is like information disclosure
* How tmux can be used to priv escalate
* A bit more on openssl and how to SSH using just a private key, which can substitute the password.

&#x20;

Security pointers:

1. Never ever disclose the private key.
2. Heartbleed is bad.
