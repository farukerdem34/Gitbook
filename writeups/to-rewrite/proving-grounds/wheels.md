# Wheels

Wheels

Sunday, June 26, 2022

12:39 PM

Nmap scan:

!\[\[60\_Wheels\_image001.png]]

&#x20;

!\[\[60\_Wheels\_image002.png]]

&#x20;

Port 80:

!\[\[60\_Wheels\_image003.png]]

&#x20;

Right away, we can take note of the stuff regarding the car-related CMS being used here, and perhaps there are exploits for this.

We can quickly register first to try and login.

!\[\[60\_Wheels\_image004.png]]

&#x20;

While we can login, we get denied to access the employee portal.

!\[\[60\_Wheels\_image005.png]]

&#x20;

Domain Name:

At the bottom of the page, we can find the email of the domain here.

!\[\[60\_Wheels\_image006.png]]

&#x20;

When we are loggin in, we can see there is some sort of Regex being used to detect the username being inputted:

!\[\[60\_Wheels\_image007.png]]

&#x20;

Might be preg\_match or something.

&#x20;

Let's review what we know:

* There's an exclusive portal.php page we should be aiming at
* This is some type of wheels service or something.
* From here, we should be thinking of a valid username and password
* Alternatively, we should be thinking of how to bypass the authentication mechanism for the page (it uses cookies to verify an administrator)

> &#x20;

I tried to brute force the username and password with hydra, and while that was running, we can look at methods to bypass the authentication methods used for this website.

&#x20;

Combinations tried: (with admin as username)

* Cewl wordlist
* Rockyou.txt with grep wheel
* Rockyou.txt with grep car
* Common car names

&#x20;

Brute forcing isn't gonna cut it. I can try to

!\[\[60\_Wheels\_image008.png]]

&#x20;

We can register as this email that we found, and then login to access the portal.php page.

!\[\[60\_Wheels\_image009.png]]

&#x20;

So now, we are presented with this weird thing.

!\[\[60\_Wheels\_image0010.png]]

&#x20;

And we get this.

This could be vulnerable to some form of SQL Injection.

When we try some funny input, we can an XML error.

!\[\[60\_Wheels\_image0011.png]]

&#x20;

So this is some kind of XXE injection.

We can certainly change this to a POST request, and begin looking for some form of XML injection. Now that we have access to this kind of apge, we should be looking for some kind of config file or something.

&#x20;

Let's think of it as the variable is called users, and there's some kind of pointer pointing to that. We just need to figure a way out to inject stuff into that entity.

&#x20;

I was googling and found that XPAth injection is the method used here injecting payloads into XML processors.

Firstly, we can find all the possible usernames.

!\[\[60\_Wheels\_image0012.png]]

&#x20;

Then we can dump passwords from this.

')] | //password%00

!\[\[60\_Wheels\_image0013.png]]

&#x20;

We can get 6 passwords for 6 users. Now we can try to SSH into each user.

The first password and username 'bob' are the ones that get us in.

&#x20;

Flag:

!\[\[60\_Wheels\_image0014.png]]

&#x20;

PE:

!\[\[60\_Wheels\_image0015.png]]

&#x20;

Within the /opt directory, there's an SUID binary.

{width="7.208333333333333in" height="2.5729166666666665in"}

&#x20;

This seems to take an input and does something with it.

We can base64 this and then use it to read files.

{width="6.0625in" height="1.3958333333333333in"}

&#x20;

This seems to open the root directory and views the files there, of which we have 2 options.

&#x20;

We can test the user input sanitisation, which is pretty weak.

{width="9.8125in" height="0.9479166666666666in"}

&#x20;

This still executes, just not with the file that we want.

I think it puts the output in /dev/null for files that do not exist.

&#x20;

Anyways, I decompiled this thing with Ghidra.

{width="7.583333333333333in" height="5.40625in"}

&#x20;

TLDR, we can see that it basically only checks for whether the string is present within the code, as it uses strstr or strchr to check for the first instance or substrings of the code.

!\[\[60\_Wheels\_image0020.png]]

&#x20;

Then it basically does this. Possibly, fgets the function could be a BOF thing, but also quite unlikely.

There could also be the potential that root has a scheduled task to do with this thing.

I eventually found out this could write files.

{width="8.635416666666666in" height="2.3541666666666665in"}

&#x20;

So I could create files as root, and this would overcome the /dev/null issue.

&#x20;

I could read the /etc/passwd file now.

{width="9.885416666666666in" height="2.3854166666666665in"}

&#x20;

Then this means I can read the /etc/shadow file and also append whatever code I want.

{width="9.885416666666666in" height="2.2916666666666665in"}

&#x20;

{width="9.822916666666666in" height="3.0104166666666665in"}

&#x20;

Then we can just su.

!\[\[60\_Wheels\_image0025.png]]

&#x20;

Flag:

!\[\[60\_Wheels\_image0026.png]]

&#x20;

Post Root:

Out of curiosity, I wanted to see if I could execute some kind of reverse shell with this binary.

And we actually can.

{width="9.729166666666666in" height="1.4583333333333333in"}

&#x20;

{width="7.375in" height="1.625in"}

&#x20;

This method would have worked if the hash cannot be cracked.

&#x20;
