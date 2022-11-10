# Frolic

Frolic

Thursday, 24 February 2022

9:41 pm

This is another Linux machine.

The target IP is 10.10.10.111.

The user IP is 10.10.16.9.

&#x20;

!\[\[57\_Frolic\_image001.png]]

Interesting!

&#x20;

Let's take a look at the ports.

Port 1880:

!\[\[57\_Frolic\_image002.png]]

Port 9999:

!\[\[57\_Frolic\_image003.png]]

&#x20;

Alright. Let's do a directory enumeration on port 9999.

Upon directory enumeration, we can see that there's an admin panel on port 9999.

!\[\[57\_Frolic\_image004.png]]

&#x20;

There's also this /backup directory here, which shows us this:

!\[\[57\_Frolic\_image005.png]]

&#x20;

!\[\[57\_Frolic\_image006.png]]

The user is **admin**.

&#x20;

I combined some directories together, and found this:

!\[\[57\_Frolic\_image007.png]]

This is something new.

&#x20;

Another log in, but that can wait.

!\[\[57\_Frolic\_image008.png]]

&#x20;

Viewing the JS reveals this.

!\[\[57\_Frolic\_image009.png]]

Alright, cool.

&#x20;

Anyways, we log in and we are presented with a cipher.

!\[\[57\_Frolic\_image0010.png]]

Some kind of cryptography going on here.

Every 5 characters is a space, hence it cannot be binary.

A bit of googling later, I found this to be Ook!

!\[\[57\_Frolic\_image0011.png]]

Doing so granted me this. Interesting.

&#x20;

This produced another weird little cipher.

!\[\[57\_Frolic\_image0012.png]]

First thing I tried was base64 decoding.

{width="11.697916666666666in" height="1.5625in"}

There was one singular file there, and this kind of told me that this bunch of text is not what it seems.

Copy over and make a file for it.

The first 2 parts there is actually, the header of a ZIP file!

&#x20;

This was indeed some kind of zip file, just encoded in base64.

&#x20;

!\[\[57\_Frolic\_image0014.png]]

Get it out and then zip2john it.

{width="10.28125in" height="3.1875in"}

Another cipher once we have decrypted this.

{width="6.9375in" height="3.7604166666666665in"}

This time in hex.

This was decrypted to base64.

&#x20;

!\[\[57\_Frolic\_image0017.png]]

Which then decrypted to whatever this was.

!\[\[57\_Frolic\_image0018.png]]

This looks to be another kind of code, but not Ook!

This one looks to be some kind of cipher, which boils down to this. It's found to be BrainFuck.

&#x20;

!\[\[57\_Frolic\_image0019.png]]

I tried this on both the Node-RED server and the playSMS, and playSMS logged me in.

!\[\[57\_Frolic\_image0020.png]]

There was an upload option within this under the send from file.

!\[\[57\_Frolic\_image0021.png]]

Quick Github search reveals that there are some form of exploits for this regarding the CSV file upload.

!\[\[57\_Frolic\_image0022.png]]

This worked! Now just time to gain a reverse shell back to me.

&#x20;

!\[\[57\_Frolic\_image0023.png]]

&#x20;

!\[\[57\_Frolic\_image0024.png]]

Finally!

With this directory, there is a config.php file.

&#x20;

{width="7.583333333333333in" height="4.84375in"}

&#x20;

I see some database creds. However, we cannot use mysql on www-data.

Let's keep this in mind first.

&#x20;

Anyways I grabbed the user flag from the user ayush.

Within this directory, there was a .binary folder which was new.

&#x20;

&#x20;

!\[\[57\_Frolic\_image0026.png]]

Heading into it, there was just one singular program called rop.

!\[\[57\_Frolic\_image0027.png]]

Everyone can execute it, and I can kind of crash it here.

Anyways I ran an ltrace against it.

&#x20;

{width="7.5in" height="1.5625in"}

I see the setuid(0) function within there, being -1.

&#x20;

I transferred the file over to my machine over netcat.

!\[\[57\_Frolic\_image0029.png]]

&#x20;

!\[\[57\_Frolic\_image0030.png]]

So now let's run this code within gdb.

&#x20;

Anyways, metasploit has a buffer overflow tool that is useful.

I created a payload of 100 characters.

!\[\[57\_Frolic\_image0031.png]]

Ran it, and took the overwritten address.

!\[\[57\_Frolic\_image0032.png]]

Took that offset address and queried it.

&#x20;

!\[\[57\_Frolic\_image0033.png]]

Cool!

So the offset is 52, now the problem is I needed to find out some memory stuff before I began to create my payload.

&#x20;

This particular ROP is called ret2libc, which would basically append some shell code at the end of the function in order to execute /bin/sh.

&#x20;

We would need to find 3 things:

1. Address of system() function
2. Address of /bin/sh
3. Address of exit function()

&#x20;

Reading [this website](https://www.ired.team/offensive-security/code-injection-process-injection/binary-exploitation/return-to-libc-ret2libc), we could use this one command to find the functions.

Unfortunately, this did not work in my favour as I did not have GDB on the target machine, else it would be very easy.

&#x20;

Let's try to find it manually!

Looking at this folder, it seems where we would find the /bin/sh file.

!\[\[57\_Frolic\_image0034.png]]

&#x20;

Scrolling down, I found the **libc.so.6** file. Let's follow the website and try to find out where /bin/sh lives.

&#x20;

Found one!

!\[\[57\_Frolic\_image0035.png]]

This means that there is an offset of 0x0015ba0b from the offset.

&#x20;

So now we just the address of the libc function.

!\[\[57\_Frolic\_image0036.png]]

Afterwards, we need to add these together. This gives us

**0xb7f74a0b**

&#x20;

Right, now we need to find the other two functions. Somehow.

With reddit, I found system()!

!\[\[57\_Frolic\_image0037.png]]

I wanted to see what other methods there were.

&#x20;

Now I need to add the same offset back to get system.

**0xb7e53da0**

&#x20;

Lastly, we need exit.

!\[\[57\_Frolic\_image0038.png]]

**0xb7e479d0**

&#x20;

This produces the 3 hex code we need.

Got this code from github.

!\[\[57\_Frolic\_image0039.png]]

Sick, now let's fill in the details and upload this onto the server.

Transferred it over HTTP to the user, and this is what I did.

&#x20;

{width="11.46875in" height="1.125in"}

&#x20;

&#x20;

Took a long time, and looked at other buffer overflow writeups to get the methodology down. Don't deal too much with them honestly. Anyways, this was real cool.

&#x20;

Learnt a lot about how to find the addresses of certain codes. That was really helpful learning how to use gdb a little as well, it's been a while.

&#x20;
