# Robust

Robust

Monday, June 27, 2022

2:27 PM

Nmap scan:

!\[\[110\_Robust \_image001.png]]

&#x20;

!\[\[110\_Robust \_image002.png]]

&#x20;

Port 80:

!\[\[110\_Robust \_image003.png]]

&#x20;

This is some SSRF.

!\[\[110\_Robust \_image004.png]]

&#x20;

!\[\[110\_Robust \_image005.png]]

&#x20;

We can use a Firefox extension to modify all the headers that are present within the machine.

!\[\[110\_Robust \_image006.png]]

&#x20;

There are few things we can do from here.

* Brute force all possible IP addresses to find out if there are ones that lead to different pages
* Brute force the usernames and passwords of this CMS
* Brute force SSH
* SQL Injection (likely to be blind)

&#x20;

There should be something hidden somewhere.

SQL injection is the only thing we can do here. So in this case, let's test out with sqlmap.

{width="5.125in" height="0.8854166666666666in"}

&#x20;

This is high likely to be a blind SQL Injection. I left this to run for a while.

{width="9.854166666666666in" height="1.75in"}

However, this appears to be some form of false positive.

The parameters do not look to be exploitable. However, this command would test every single parameter, from the Cookie to everything within the request.

&#x20;

Let's take a look at the login.php page again.

!\[\[110\_Robust \_image009.png]]

&#x20;

When we send another directory, it refers us to this login.php using the Location header.

From here, we can gobust the directories using wfuzz.

&#x20;

!\[\[110\_Robust \_image0010.png]]

&#x20;

!\[\[110\_Robust \_image0011.png]]

&#x20;

There's a home.php.

!\[\[110\_Robust \_image0012.png]]

&#x20;

We can see how there is a Location header that forces us to return to the login.php page.

Now we should be considering options to bypass this location page sort of.

&#x20;

Using a hint, I could see that we were able to intercept server responses and remove headers.

!\[\[110\_Robust \_image0013.png]]

&#x20;

!\[\[110\_Robust \_image0014.png]]

&#x20;

With this, we are able to visit the home.php page.

!\[\[110\_Robust \_image0015.png]]

&#x20;

With this, we can see that there is a firstname and lastname search query here.

If we key in the 1=1 thing, we would be able to gain all of the different employees.

!\[\[110\_Robust \_image0016.png]]

&#x20;

And we can see how there's this user called...Hidden.

If we were to use the 1=2 function instead, we would not see the hidden user.

&#x20;

We can then abuse this through the use of UNION injection.

!\[\[110\_Robust \_image0017.png]]

&#x20;

We can confirm this by using some form of random input.

!\[\[110\_Robust \_image0018.png]]

&#x20;

I tried to enumerate the database, but was unable to find some form of table name or something.

Then we can just randomly guess a table name.

&#x20;

!\[\[110\_Robust \_image0019.png]]

&#x20;

With this password, we can SSH into the machine as 'jeff'.

&#x20;

Flag:

!\[\[110\_Robust \_image0020.png]]

0fdd0a84362a06c496204bf9c2c64a19

&#x20;

PE:

We can see that Jeff is allowed to do some stuff.

!\[\[110\_Robust \_image0021.png]]

&#x20;

We can also find the password of the administrator of the website.

{width="8.416666666666666in" height="3.15625in"}

&#x20;

WinPEAS:

&#x20;

!\[\[110\_Robust \_image0023.png]]

&#x20;

From here, we can begin enumerating this for passwords or DLL hijacking due to that SHUTDOWN privilege.

!\[\[110\_Robust \_image0024.png]]

This has nothing but my own powershell commands.

&#x20;

Within the home directory of jeff, we can do dir /s and view all the files that come with it.

!\[\[110\_Robust \_image0025.png]]

&#x20;

There's these files here.

Sticky notes are generally quite vulnerable.

We can view these files using cat or type and see that there's a password.

!\[\[110\_Robust \_image0026.png]]

&#x20;

And now we can SSH in as the administrator.

&#x20;

Flag:

!\[\[110\_Robust \_image0027.png]]

&#x20;

Quite unrealistic for the PE, but hey, I'll take it.

&#x20;
