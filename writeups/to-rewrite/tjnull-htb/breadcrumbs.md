# BreadCrumbs

BreadCrumbs

Saturday, 12 March 2022

6:03 pm

This is a Windows machine.

The target IP is 10.10.10.

My IP is 10.10.16.9.

&#x20;

Interesting box.

&#x20;

!\[\[80\_BreadCrumbs\_image001.png]]

&#x20;

!\[\[80\_BreadCrumbs\_image002.png]]

&#x20;

Website:

!\[\[80\_BreadCrumbs\_image003.png]]

&#x20;

This is a library or book website.

We can view books through the search.

!\[\[80\_BreadCrumbs\_image004.png]]

When viewing the request in burp, we see that it returns in a JSON format.

&#x20;

!\[\[80\_BreadCrumbs\_image005.png]]

When no results are found, then it would return an empty array.

&#x20;

Right, so let's take a look at the HTTPS port.

Looks to be the same website though...

The certificate does not tell us anything.

&#x20;

Ran a gobuster with a php extension, because this was indeed a PHP server.

While that was running, I managed to get some stuff out of the website.

&#x20;

When viewing a book, this is what would happen:

!\[\[80\_BreadCrumbs\_image006.png]]

&#x20;

!\[\[80\_BreadCrumbs\_image007.png]]

Within Burp, this is what the request looks like:

!\[\[80\_BreadCrumbs\_image008.png]]

&#x20;

When altering the book parameter with a ' at the end, it reveals that this gets the books using file\_get\_contents() function.

!\[\[80\_BreadCrumbs\_image009.png]]

&#x20;

Perhaps we can try some directory traversal or escaping.

!\[\[80\_BreadCrumbs\_image0010.png]]

&#x20;

The directory enumeration reveals the db directory.

&#x20;

!\[\[80\_BreadCrumbs\_image0011.png]]

This does not reveal anything, and I suspect there to be some other forms of stuff we can get from this.

&#x20;

I played around with this, and did some directory traversal.

&#x20;

!\[\[80\_BreadCrumbs\_image0012.png]]

This returned something!

&#x20;

Now, we know that we can indeed read any file from this. First target was that db.php file to see what contents was inside of it.

We can even see the insecure function, and how it does not protect against directory traversal.

&#x20;

From this, I was able to read the db.php file.

&#x20;

!\[\[80\_BreadCrumbs\_image0013.png]]

Right, now we have this SQLI password and host, which is great.

&#x20;

Now, we need to find somewhere to use this.

Within the gobuster, we can see that there is the portal directory.

&#x20;

!\[\[80\_BreadCrumbs\_image0014.png]]

Visiting it redirects us to this website.

!\[\[80\_BreadCrumbs\_image0015.png]]

The credentials we have does not help us here...

&#x20;

Playing around with the login request in Burp, I saw this:

!\[\[80\_BreadCrumbs\_image0016.png]]

Right, so we have a new directory to visit.

!\[\[80\_BreadCrumbs\_image0017.png]]

&#x20;

I see that this PHP code is using ===, meaning the owner is aware of magic hashes.

Looking at the code, I also saw this little string here.

&#x20;

!\[\[80\_BreadCrumbs\_image0018.png]]

There's a JWT token here, and this gives me good ideas.

&#x20;

We already have exposed the key, and we just need to find the payload of which is being encrypted.

!\[\[80\_BreadCrumbs\_image0019.png]]

&#x20;

This could lead to some token stealing, whereby we can spoof the token easily.

The next interesting part is that this uses another cookie.php.

&#x20;

!\[\[80\_BreadCrumbs\_image0020.png]]

&#x20;

{width="11.729166666666666in" height="1.65625in"}

Right, we have more information.

&#x20;

The part we want is this:

!\[\[80\_BreadCrumbs\_image0022.png]]

&#x20;

This chunk essentially means

$key **=** "s4lTy\_stR1nG\_".$username\[$seed]."(!528.\\/9890";

&#x20;

Afterwards this session key used is this:

!\[\[80\_BreadCrumbs\_image0023.png]]

So it's paul.md5 of whatever key is there.

&#x20;

Looking at the code, it seems that the key is dependent on the username length.

Additionally, the login page has this helper function, which reveals some usernames to use:

&#x20;

!\[\[80\_BreadCrumbs\_image0024.png]]

&#x20;

Okay, we have some users. Looking at the login page, we can see that it uses the authController to verify the user logging in.

I created an account on the signup page.

&#x20;

!\[\[80\_BreadCrumbs\_image0025.png]]

&#x20;

And we could log in:

!\[\[80\_BreadCrumbs\_image0026.png]]

Viewing the user management part there, we can see all the different users.

&#x20;

!\[\[80\_BreadCrumbs\_image0027.png]]

&#x20;

The File management part didn't work, so I looked at burp to get the directory it was using. Turns out it was redirecting us back to the main dashboard.

!\[\[80\_BreadCrumbs\_image0028.png]]

&#x20;

&#x20;

Viewing the source code:

{width="8.854166666666666in" height="0.8125in"}

It was verifying that I do not have the correct content needed to access this.

From here, I could determine that the user had to be paul.

&#x20;

Additionally, there was something interesting with the tokens I was given in the file management request.

&#x20;

!\[\[80\_BreadCrumbs\_image0030.png]]

Seems that there's a very nice PHPSESSID and JWT token there, and the next step would be to spoof these somehow.

&#x20;

We know the user we need to be, which is paul.

&#x20;

Using some online tools, I can change the JWT token to who I need it to be, using the secret key that was given to me.

!\[\[80\_BreadCrumbs\_image0031.png]]

&#x20;

One down, now we need to figure out that PHPSESSID.

&#x20;

Looking at the code, the first thing I tried was to change the front part to my username, assuming that the SESSID cookie does not change for each of them first. This was true because the key was static, in which it always used back the same key which was appended at the end.

&#x20;

Changed the front part of my token to 'paul(hex)' and the token to the JWT token, and I found out we could upload .zip files here.

&#x20;

!\[\[80\_BreadCrumbs\_image0032.png]]

&#x20;

Right, so we need some kind of webshell that allows us to upload zip files.

Let's create a simple webshell and attempt to upload it.

&#x20;

!\[\[80\_BreadCrumbs\_image0033.png]]

This resulted in some kind of error:

&#x20;

!\[\[80\_BreadCrumbs\_image0034.png]]

So we know that there is some WAF or something. Taking a look at the request:

&#x20;

!\[\[80\_BreadCrumbs\_image0035.png]]

There's some task there trying to make it a .zip file. Let's change that .zip to .php.

&#x20;

Still doesn't work.

There's some other WAF in play here...

So, I googled other forms of php web shells, and came across one that does shell\_exec to bypass the checks.

&#x20;

Works!

!\[\[80\_BreadCrumbs\_image0036.png]]

&#x20;

But I'm unable to execute my code there. Perhaps I should use a different code:

!\[\[80\_BreadCrumbs\_image0037.png]]

This worked. Also the last part is the name of the code we are going in.

&#x20;

We already know that the webshell is uploaded to ../uploads/shell.php.

We uploaded it properly, but the code has an error it seems:

&#x20;

!\[\[80\_BreadCrumbs\_image0038.png]]

At least we know we're right.

&#x20;

Eventually I tried the original line and it worked?

&#x20;

!\[\[80\_BreadCrumbs\_image0039.png]]

Right, so now we have RCE.

&#x20;

The next step, which is the simplest is just to take netcat and upload it onto the server.

!\[\[80\_BreadCrumbs\_image0040.png]]

&#x20;

Now, load it!

!\[\[80\_BreadCrumbs\_image0041.png]]

&#x20;

!\[\[80\_BreadCrumbs\_image0042.png]]

Great!

&#x20;

There's one other user on this box:

!\[\[80\_BreadCrumbs\_image0043.png]]

&#x20;

Going out of the directory, we see this:

!\[\[80\_BreadCrumbs\_image0044.png]]

There's this user data, which I'm particularly curious about.

&#x20;

Within it, there's one file that stands out:

!\[\[80\_BreadCrumbs\_image0045.png]]

From this, we get...creds?

&#x20;

!\[\[80\_BreadCrumbs\_image0046.png]]

&#x20;

I was thinking about how to access this user, until I realised SSH was open on this machine.

And we're in!

{width="6.0in" height="7.5in"}

Within her Desktop, there was another todo.html located there.

&#x20;

!\[\[80\_BreadCrumbs\_image0048.png]]

&#x20;

!\[\[80\_BreadCrumbs\_image0049.png]]

This was a straight hint, and there are passwords there.

If you store credentials in plaintext, no wonder you don't get promoted.

&#x20;

A bit of googling reveals that the files are stored here:

!\[\[80\_BreadCrumbs\_image0050.png]]

&#x20;

So, we can head to here:

!\[\[80\_BreadCrumbs\_image0051.png]]

We need these files somehow.

SCP wasn't working to transfer these files, so let's do it using smb.

&#x20;

!\[\[80\_BreadCrumbs\_image0052.png]]

&#x20;

Now we have these files here.

&#x20;

Let's take a look at them one by one.

&#x20;

!\[\[80\_BreadCrumbs\_image0053.png]]

The User table was not useful, while the Note tables was:

&#x20;

!\[\[80\_BreadCrumbs\_image0054.png]]

Great, another password.

&#x20;

Let's use smbmap now.

{width="10.322916666666666in" height="2.5in"}

The difference was, now I have access to the development file.

Naturally, the box moved the admin password elsewhere.

{width="2.8854166666666665in" height="1.03125in"}

&#x20;

There was one file within it.

&#x20;

!\[\[80\_BreadCrumbs\_image0057.png]]

&#x20;

Looking at the code, this was an ELF file, meaning I could execute it somehow.

&#x20;

!\[\[80\_BreadCrumbs\_image0058.png]]

&#x20;

!\[\[80\_BreadCrumbs\_image0059.png]]

We needed some kind of key.

&#x20;

IDA time.

Looking at the code, there seems to be this website there.

&#x20;

!\[\[80\_BreadCrumbs\_image0060.png]]

&#x20;

One check on the ports that were open on the machine, it seems that this passmanager thing could not be accessed by me. I then proceeded to try some SSH tunneling, finding it odd that the website even gave me the port number there.

&#x20;

Verified by the machine itself that the port was indeed running:

!\[\[80\_BreadCrumbs\_image0061.png]]

&#x20;

!\[\[80\_BreadCrumbs\_image0062.png]]

&#x20;

!\[\[80\_BreadCrumbs\_image0063.png]]

Alright, something.

{width="5.3125in" height="0.6666666666666666in"}

This were the things we have to append to the back.

&#x20;

!\[\[80\_BreadCrumbs\_image0065.png]]

Right, so we get this key.

&#x20;

Great...

&#x20;

But from here, I tried some SQL injection, because I can. If the table there is an SQL command, this means I can append more behind.

This was a direct SQL command, and I can see how it works.

&#x20;

Knowing that there are probably lots of tables included within there, I just ran a UNION select command to check what it would return:

!\[\[80\_BreadCrumbs\_image0066.png]]

Right.

&#x20;

!\[\[80\_BreadCrumbs\_image0067.png]]

Found out the databases present.

&#x20;

Now, let's go into that.

&#x20;

!\[\[80\_BreadCrumbs\_image0068.png]]

&#x20;

Now column names.

!\[\[80\_BreadCrumbs\_image0069.png]]

&#x20;

Let's now select everything and display it for all.

&#x20;

!\[\[80\_BreadCrumbs\_image0070.png]]

I know what the key is, so now we have this which is AES?

&#x20;

Got lazy and didn't want to look up a script:

!\[\[80\_BreadCrumbs\_image0071.png]]

&#x20;

!\[\[80\_BreadCrumbs\_image0072.png]]

&#x20;

Finally:

{width="6.15625in" height="2.3854166666666665in"}

&#x20;
