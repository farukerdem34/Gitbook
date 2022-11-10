# Injecto

Injecto

Monday, 21 March 2022

10:08 am

Linux machine with these services running on it. This machine involved PHP deserialisation, whereby we there was an LFI due to some PHP stuff. This was something I never knew...

&#x20;

I took the walkthrough for this because I was unsure of how to even find this vulnerability.

&#x20;

{width="5.6875in" height="1.125in"}

Through the LFI, we can take the find the source code of the website, and we can see this function. Unserializing input like this is dangerous, as it can lead to RCEs.

&#x20;

We can see that the web page takes in one parameter, and from there it deserializes it, allowing for a possibility of an RCE.

From here, we can actually inject some code in the form of cmd.php.

&#x20;

Right, so from here, we can craft a quick deserialisation objection injection. This would be able to execute commands on our behalf. For us, we can change this system into some cmd.php shell, and get it to write to a file that is to be stored within the database.

!\[\[25\_Injecto\_image002.png]]

&#x20;

!\[\[25\_Injecto\_image003.png]]

This was taken from the answer.

&#x20;

From this, we can then submit a GET request with the values\_submit parameter equating to this. This would allow us to basically get a PHP injection vulnerability. From here, we can grab a shell.sh file and set up a quick python server for us to use.

&#x20;

This is how we get our reverse shell.

!\[\[25\_Injecto\_image004.png]]

&#x20;

From here, the privilege escalation would involve some SUID abuse.

!\[\[25\_Injecto\_image005.png]]

We can use curl on this machine, and what we can do with this is basically rewrite the /etc/passwd file to add our own root user.

&#x20;

!\[\[25\_Injecto\_image006.png]]

Copy this user into our the malicious passwd file, and then from there we can copy the /etc/passwd file on the machine and begin our curl.

Set up a python server and then curl the passwd file with the -O flag to the /etc/passwd file.

&#x20;

From there, we can add this user 'hacker' which has root access and a password of our choosing (this one is hello).

!\[\[25\_Injecto\_image007.png]]

&#x20;

!\[\[25\_Injecto\_image008.png]]

&#x20;
