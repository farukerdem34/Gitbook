# Megavolt

Megavolt

Wednesday, 30 March 2022

3:47 pm

A Linux machine with these ports:

!\[\[42\_Megavolt\_image001.png]]

&#x20;

&#x20;

Port 80 brings us to this ticket support center.

!\[\[42\_Megavolt\_image002.png]]

&#x20;

Quite familiar, as I've seen this before in HTB.

We can create an account and open a ticket here.

!\[\[42\_Megavolt\_image003.png]]

&#x20;

We would get this reply, and also the user does not seem to be clicking links.

Since there is a user that is viewing our tickets, perhaps we could steal his cookie through the use of XSS.

!\[\[42\_Megavolt\_image004.png]]

&#x20;

There were multiple vulnerabilities in this, and I wanted to test and see what I could find.

Based on some research, it seems that the upload attachment portion of the website is vulnerable to XSS.

[https://www.exploit-db.com/exploits/47224](https://www.exploit-db.com/exploits/47224)

However, the exploit requires we be able to login as another user to view the exploit, hence this does not give us the cookie automatically.

&#x20;

As such, we can change the HTML recommended to something that would connect to our machine remotely.

!\[\[42\_Megavolt\_image005.png]]

&#x20;

Then, we would get this on a listener port.

!\[\[42\_Megavolt\_image006.png]]

&#x20;

We can see a cookie value as the OSTSESSID. This should be the administrator's cookie, as he's the one that's going around closing the tickets being created.

&#x20;

With the administrator cookie, we can try to login as him. We can authenticate as this user accordingly.

We can run this within the console of the web browser.

Document.cookie = 'OSTSESSID=cookie'

&#x20;

Then, we can head to /scp to log in as alfred.

{width="12.114583333333334in" height="3.7604166666666665in"}

&#x20;

In the admin panel under settings, we can see the version that is being used here.

!\[\[42\_Megavolt\_image008.png]]

&#x20;

We can gain a reverse shell through the use of this:

[https://github.com/Und3r-r00t/osticket-shell](https://github.com/Und3r-r00t/osticket-shell)

&#x20;

!\[\[42\_Megavolt\_image009.png]]

&#x20;

Exploit:

When we upload any ticket with our shell, we can view it using the administrator account. We can also edit the settings to store files uploaded.

{width="10.416666666666666in" height="2.625in"}

&#x20;

!\[\[42\_Megavolt\_image0011.png]]

&#x20;

Afterwards, we can change the settings to store files within the machine itself.

!\[\[42\_Megavolt\_image0012.png]]

&#x20;

Exploit:

{width="10.072916666666666in" height="2.1979166666666665in"}

&#x20;

!\[\[42\_Megavolt\_image0014.png]]

&#x20;

!\[\[42\_Megavolt\_image0015.png]]

&#x20;

!\[\[42\_Megavolt\_image0016.png]]

&#x20;

This would be our key that we need to use for the exploit.

The script can then guess the file where it's at.

{width="8.40625in" height="2.53125in"}

&#x20;

We can take the hash of this and then proceed to brute force the files.

!\[\[42\_Megavolt\_image0018.png]]

&#x20;

Then, when executed, it would brute force every single possible file that can exist and eventually give us a shell by executing our PHP reverse shell.

{width="8.71875in" height="2.4166666666666665in"}

&#x20;

Flag:

!\[\[42\_Megavolt\_image0020.png]]

13108b75d5242040d28b0fc625368010

&#x20;

PE:

!\[\[42\_Megavolt\_image0021.png]]

&#x20;

With tee, we can basically write any files within the httpd file.

Since there is a wilcard, which is always vulnerable, it means it does not check for the end of the file path.

&#x20;

We can do something like this to echo stuff into the /etc/passwd file (be careful, wrongly typing this would wipe the entire passwd file and we have to restart.

&#x20;

!\[\[42\_Megavolt\_image0022.png]]

&#x20;

Then just su.

!\[\[42\_Megavolt\_image0023.png]]

&#x20;

Flag:

!\[\[42\_Megavolt\_image0024.png]]

ea438b6d1a07a617aa413bb462a5a90e

&#x20;

&#x20;
