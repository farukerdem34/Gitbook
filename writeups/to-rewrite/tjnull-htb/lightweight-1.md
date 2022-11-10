# Lightweight

Lightweight

Sunday, 6 March 2022

3:30 pm

This is a Linux machine.

The target IP is 10.10.10.119.

My IP is 10.10.16.2.

&#x20;

Enumeration time.

Let's check all the ports on this machine.

&#x20;

!\[\[70\_Lightweight\_image001.png]]

&#x20;

!\[\[70\_Lightweight\_image002.png]]

&#x20;

Website:

!\[\[70\_Lightweight\_image003.png]]

Notice the ban? Well, this website protects against brute forcing, it seems...

Reset the machine to get rid of the ban faster. We need to go slow on this, and we cannot be running any form of brute forcing.

&#x20;

This is what the page is supposed to look like:

!\[\[70\_Lightweight\_image004.png]]

Perhaps I should have read it first before getting banned.

&#x20;

When viewing the user directory, we are greeted with this:

!\[\[70\_Lightweight\_image005.png]]

&#x20;

I have a feeling this is a rabbit hole. So let's take a look at that LDAP server instead.

&#x20;

!\[\[70\_Lightweight\_image006.png]]

&#x20;

!\[\[70\_Lightweight\_image007.png]]

A hash!

This seems to be for ldapuser1.

&#x20;

!\[\[70\_Lightweight\_image008.png]]

Ldapuser2 has one too.

Well, these hashes don't crack so let's try using SSH into our IP address account into the machine.

&#x20;

Let's use our IP address as the username and password to get into the box and see what we can sniff out.

&#x20;

!\[\[70\_Lightweight\_image009.png]]

&#x20;

{width="6.0625in" height="0.6979166666666666in"}

Interestingly, there are other users that are also present on the box.

&#x20;

Hmm, this gives me nothing. There's really nothing much else to evaluate, so I resorted to using Wireshark to sniff something.

Referred to a guide in order to understand what I was doing.

&#x20;

Turns out we need to use this command:

tcpdump -i lo -nnXs 0 'port 389'

&#x20;

To listen on the SSH user, in order to sniff out any authentication packets that are being sent between the server and LDAP when we check the status.php file.

&#x20;

We would get this:

{width="7.90625in" height="2.3229166666666665in"}

That there is the password for ldapuser2!

&#x20;

Su and change to it.

{width="4.895833333333333in" height="0.9791666666666666in"}

There are a few interesting files within this directory.

!\[\[70\_Lightweight\_image0013.png]]

There's this back up file which I want to transfer back to my machine.

Looks suspicious. The rest I don't really care about.

&#x20;

Used base64 to transfer it over to my machine.

!\[\[70\_Lightweight\_image0014.png]]

Now we can 7ztojohn this thing first, because when trying to extract it, we need a password.

&#x20;

Then john it to crack and let's wait for to work.

{width="9.729166666666666in" height="3.1666666666666665in"}

Right, now let's decrypt it.

{width="12.229166666666666in" height="5.864583333333333in"}

Found this within one of the files called status.php.

&#x20;

{width="5.28125in" height="0.8229166666666666in"}

Su time.

&#x20;

Seems that there's another pcap file and stuff like that.

&#x20;

!\[\[70\_Lightweight\_image0018.png]]

The pcap and the PHP file are not interesting, but the openssl file is pretty interesting.

&#x20;

Based on GTFObins, we can actually use this to read files.

{width="7.78125in" height="0.96875in"}

Using this, we can take the hashes from the /etc/shadow file and crack the password from there.

&#x20;

!\[\[70\_Lightweight\_image0020.png]]

Cracking this does not work.

&#x20;

So let's try changing the password of root then!

!\[\[70\_Lightweight\_image0021.png]]

Let's replace that with the current one.

&#x20;

!\[\[70\_Lightweight\_image0022.png]]

&#x20;

Now we need to upload this file back into our victim machine.

!\[\[70\_Lightweight\_image0023.png]]

Then change the current passwords file to our malicious one. Afterwards, Su using our password of hello

&#x20;

{width="7.114583333333333in" height="1.65625in"}

There we go.
