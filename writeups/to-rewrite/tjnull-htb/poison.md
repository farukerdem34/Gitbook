# Poison

Poison

Tuesday, 8 February 2022

10:39 am

This machine IP is 10.10.10.84, and it is a Linux machine.

My IP is 10.10.16.5.

&#x20;

Enumeration. Fast scan tells me that there is a http and ssh port open.

Detailed scan tells me the following.

&#x20;

!\[\[23\_Poison\_image001.png]]

&#x20;

FreeBSD, take note!

The web page seems to be a temporary website to test local .php scripts...so in this case we need to check on the php scripts that are present on the thing.

&#x20;

!\[\[23\_Poison\_image002.png]]

&#x20;

There's a phpinfo.php, which reveals a lot of information about the OS. In this case it is FreeBSD Poison 11.1.

!\[\[23\_Poison\_image003.png]]

&#x20;

The main page seems to use browse.php to read files and stuff. When we try to enter wrong stuff, we get this:

!\[\[23\_Poison\_image004.png]]

&#x20;

Looking at the website, there is a listfiles.php, which shows us this.

{width="11.479166666666666in" height="1.34375in"}

This is showing us the files present in the directory, which is interesting because there's a text file in it. Entering the name reveals some base64 code.

&#x20;

!\[\[23\_Poison\_image006.png]]

&#x20;

Decrypted, it gives us this: **Charix!2#4%6&8(0**

&#x20;

Looks to be a password. I guessed that the user was Charix, and I got a shell.

!\[\[23\_Poison\_image007.png]]

&#x20;

There's an interesting secret.zip within the directory.

!\[\[23\_Poison\_image008.png]]

Seems to be needing a passphrase in order to get it out.

&#x20;

I looked around for a long time and was unable to find anything, and resorted to a walkthrough.

&#x20;

I transferred the file over to my kali machine as such:

!\[\[23\_Poison\_image009.png]]

&#x20;

{width="4.4375in" height="0.625in"}

&#x20;

This would be cracked with charix password and reveal some 'secret' file that has some weird characters in it.

&#x20;

So what we're supposed to do is SSH Tunneling.

&#x20;

This is because when viewing the open ports, we can see that port 5801 and 5901 are open.

!\[\[23\_Poison\_image0011.png]]

&#x20;

&#x20;

These two ports are for VNC, which is basically for remote desktop controller over a browser.

&#x20;

When we need to do is just do an SSH tunnel from there to our machine and we can access the root user.

!\[\[23\_Poison\_image0012.png]]

&#x20;

Do so, and then we can use vncviewer to view the thing, remember to take along our password that we decrypted.

!\[\[23\_Poison\_image0013.png]]

&#x20;

And this will spawn.

!\[\[23\_Poison\_image0014.png]]

&#x20;

SSH tunneling is something I need to get used to. It is a method of which we transport arbitrary data over an encrypted SSH connection.

&#x20;

So basically, this allows for us to bind a local port to the port on the machine. This is used when that open port is hiding behind something like a firewall and cannot be accessed normally (nmap could not detect it).

&#x20;

The SSH tunnel would allow for traffic between the local and remote machines.

&#x20;

The command is as follows:

**Ssh -L VICTIM\_PORT:localhost:ATTACKER\_PORT user@victim**

&#x20;

This would open the ATTCKER\_PORT and it would listen for any traffic coming from the VICTIM\_PORT. This is useful as it allows traffic to flow from our port to their port.

&#x20;

Hence when we do the vncviewer, we use our local machine's port to connect to that port.

&#x20;

When needing to do SSH tunneling, keep in mind the command **netstat -an** as it reveals all the open listening ports on the victim machine.
