# Thor

Thor

Tuesday, July 12, 2022

10:59 AM

Nmap scan:

!\[\[80\_Thor \_image001.png]]

&#x20;

!\[\[80\_Thor \_image002.png]]

&#x20;

Port 80:

!\[\[80\_Thor \_image003.png]]

&#x20;

This is some Avengers related website.

Trying to send some kind of contact note would result in this.

!\[\[80\_Thor \_image004.png]]

&#x20;

We can see that it runs on PHP for the backend, and takes parameters through GET requests.

&#x20;

AT the bottom, we see some kind of CMS.

!\[\[80\_Thor \_image005.png]]

&#x20;

Litespeed webserver is something that has some form of exploits.

&#x20;

Gobusting:

!\[\[80\_Thor \_image006.png]]

&#x20;

&#x20;

We can see this docs and cgi-bin thing.

&#x20;

/docs:

!\[\[80\_Thor \_image007.png]]

&#x20;

Gives us the version of the CMS that is running, of which case it is vulnerable to exploits.

&#x20;

From here, we can check on potential exploits.

{width="9.322916666666666in" height="2.2916666666666665in"}

&#x20;

There are command injection exploits for this that require credentials.

[https://www.exploit-db.com/exploits/49556](https://www.exploit-db.com/exploits/49556)

&#x20;

This would require us to access the CMS on port 7080.

&#x20;

We can enumerate this page a bit more to find some potential usernames.

!\[\[80\_Thor \_image009.png]]

&#x20;

We can take username.py to generate a quick wordlist of usernames for this.

{width="6.125in" height="0.84375in"}

&#x20;

Port 7080:

!\[\[80\_Thor \_image0011.png]]

&#x20;

We can attempt to brute force this login page.

&#x20;

Now, we only know that the user is named Jane Foster, which is a clue to a possible password. The username is likely something like 'admin', and the password is tied to the username of the user.

&#x20;

I tried with a dozen usernames, and eventually found that **admin:Foster2020** works.

!\[\[80\_Thor \_image0012.png]]

&#x20;

We can see the version here.

&#x20;

Let's grab a public exploit:

!\[\[80\_Thor \_image0013.png]]

&#x20;

When checking the code, we can change the command to have the correct IP address in order to gain a reverse shell.

!\[\[80\_Thor \_image0014.png]]

&#x20;

!\[\[80\_Thor \_image0015.png]]

&#x20;

!\[\[80\_Thor \_image0016.png]]

&#x20;

Fixing shell:

!\[\[80\_Thor \_image0017.png]]

&#x20;

!\[\[80\_Thor \_image0018.png]]

&#x20;

We are part of the shadow group, so let's read /etc/shadow.

!\[\[80\_Thor \_image0019.png]]

&#x20;

!\[\[80\_Thor \_image0020.png]]

&#x20;

Then crack hashes.

Likely, the hash from thor will crack and root won't. Else the box would be too easy.

&#x20;

We can crack the hash of thor.

!\[\[80\_Thor \_image0021.png]]

&#x20;

!\[\[80\_Thor \_image0022.png]]

SSH in as thor.

&#x20;

Flag:

!\[\[80\_Thor \_image0023.png]]

&#x20;

PE:

!\[\[80\_Thor \_image0024.png]]

&#x20;

We can run systemctl and restart the webmin instance, which gives me ideas on how to privilege escalate.

Editing the service file of webmin would allow us to basically gain RCE as root.

&#x20;

Finding webmin process:

!\[\[80\_Thor \_image0025.png]]

&#x20;

Interesting because there's a script running and it seems to be editing the miniserv.conf file. Root is also the one running this.

Perhaps we can run this script and allow ourselves to do something with it.

&#x20;

Anyways, we cannot run this script and change the password of the instance, but we can however run the script using the previous exploit, which basically assigns us a group.

!\[\[80\_Thor \_image0026.png]]

&#x20;

We can see that the bin group can edit the conf file.

&#x20;

Then we can see within the files, there is this changepass.pl.

!\[\[80\_Thor \_image0027.png]]

&#x20;

Executable by all as well.

We just need to use this script to change the password of the webmin instance, and then exploit it for RCE as root.

&#x20;

Getting shell as bin:

!\[\[80\_Thor \_image0028.png]]

&#x20;

!\[\[80\_Thor \_image0029.png]]

&#x20;

Then we can change the password of the webmin instance.

!\[\[80\_Thor \_image0030.png]]

&#x20;

Then, we can restart the webmin instance using the sudo privileges.

!\[\[80\_Thor \_image0031.png]]

&#x20;

Afterwards, we can just get the webmin package RCE exploit from MSF.

!\[\[80\_Thor \_image0032.png]]

&#x20;

!\[\[80\_Thor \_image0033.png]]

&#x20;

We have RCE as root.

&#x20;

Flag:

!\[\[80\_Thor \_image0034.png]]

&#x20;
