# Keyvault

Keyvault

Wednesday, June 29, 2022

8:22 PM

Nmap scan:

!\[\[64\_Keyvault \_image001.png]]

&#x20;

&#x20;

!\[\[64\_Keyvault \_image002.png]]

&#x20;

Port 80:

!\[\[64\_Keyvault \_image003.png]]

&#x20;

Just a regular old page, nothing much.

&#x20;

Port 8080:

!\[\[64\_Keyvault \_image004.png]]

&#x20;

We can attempt to login with admin:admin and we would view this.

Which is weird.

&#x20;

&#x20;

!\[\[64\_Keyvault \_image005.png]]

&#x20;

There is a git repository which we can check out. After downloading, we can view the logs.

{width="6.875in" height="1.8020833333333333in"}

&#x20;

From here, we can view all the files that have been changed.

{width="6.96875in" height="3.46875in"}

&#x20;

!\[\[64\_Keyvault \_image008.png]]

We can learn that the password and username we guessed are correct.

Then we can figure out the parameters that are needed for this.

!\[\[64\_Keyvault \_image009.png]]

&#x20;

The weird part is that well, the username is different. In the git logs it uses Ray, but on the website it uses Chris.

What a weird application.

&#x20;

There should be some form of code injection somewhere, because this website does nothing otherwise.

&#x20;

We can see that what is being done is quite unique, it is getting this hm thing and then hashing it to compare.

!\[\[64\_Keyvault \_image0010.png]]

&#x20;

I don't know what else it was supposed to do, but we are able to control this 'token' variable.

This looks like a hash extender attack, because there is a $secret variable and there is a token which is a user controlled input.

&#x20;

This is a good resource to understand the bugs:

[https://www.securify.nl/blog/spot-the-bug-challenge-2018-warm-up/](https://www.securify.nl/blog/spot-the-bug-challenge-2018-warm-up/)

&#x20;

However, after we send this in, we get nothing...?

!\[\[64\_Keyvault \_image0011.png]]

&#x20;

So basically we have a valid thing here for the hmac.php, and then we can proceed to index.php.

&#x20;

!\[\[64\_Keyvault \_image0012.png]]

&#x20;

We can now SSH in as ray.

&#x20;

Flag:

!\[\[64\_Keyvault \_image0013.png]]

&#x20;

PE:

Within the /opt directory, there's this binary that is owned by root.

!\[\[64\_Keyvault \_image0014.png]]

&#x20;

I still ran pspy and linpeas to check if there are any other vectors for PE.

&#x20;

When checking this binary, we can see that it does something like this.

!\[\[64\_Keyvault \_image0015.png]]

&#x20;

When run pspy, we can see this.

!\[\[64\_Keyvault \_image0016.png]]

&#x20;

This thing does an su as the root user, and then starts apaceh2 as expected. It does prompt for a password or something, which I think can be taken out if we manage to reverse engineer the file.

&#x20;

I transferred the file to my machine using nc.

&#x20;

Then I used some Ghidra. We should be finding a kind of password that is being entered when we run this binary. Because the binary runs SU, we should be able to see some form of password being entered when the binary is executed, hence the password prompt. This would explain why the processes being executed are by the root user.

&#x20;

Opening the program in Ghidra, we are able to see that it has been modified such that it is harder to find out what it really does.

&#x20;

{width="7.927083333333333in" height="2.65625in"}

&#x20;

This is the main function, whereby the thunk\_FUN should be main().

&#x20;

When looking at the main function, we can see that there is this PYInstaller thing and also MEIPASS2.

!\[\[64\_Keyvault \_image0018.png]]

&#x20;

Interesting because there are PyInstaller Extractors online, and we may need to use this on it.

&#x20;

Taking this script here:

[https://github.com/extremecoders-re/pyinstxtractor/blob/master/pyinstxtractor.py](https://github.com/extremecoders-re/pyinstxtractor/blob/master/pyinstxtractor.py)

!\[\[64\_Keyvault \_image0019.png]]

&#x20;

We can now view the files that are within this.

Once we extracted it, we can view the code.

!\[\[64\_Keyvault \_image0020.png]]

&#x20;

Then, we can use the Python Decompiler to further break down the .pyc files, which would be broken down into source code.

Alternatively, we can just read the apache-restart.pyc file using strings.

&#x20;

!\[\[64\_Keyvault \_image0021.png]]

&#x20;

We can see this base64 string here, and when decoded, gives this string.

!\[\[64\_Keyvault \_image0022.png]]

&#x20;

This is the password for root.

&#x20;

Su:

!\[\[64\_Keyvault \_image0023.png]]

&#x20;

Flag:

!\[\[64\_Keyvault \_image0024.png]]

6ee9f2a257aad749e0ce8747c214469e

&#x20;

Ghidra OP.

&#x20;
