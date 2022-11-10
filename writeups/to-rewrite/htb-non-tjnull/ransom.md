# Ransom

Ransom

Tuesday, 29 March 2022

2:15 pm

A Linux machine with these ports:

&#x20;

!\[\[12\_Ransom\_image001.png]]

&#x20;

When looking at the HTTP port, we have a login here.

!\[\[12\_Ransom\_image002.png]]

&#x20;

When looking at the page source, we can see this little snippet.

!\[\[12\_Ransom\_image003.png]]

&#x20;

Ran a gobuster on the /api directory, but all of it seems to return us back to the login directory.

&#x20;

Upon looking at the web request, it seems that it has a laravel\_session cookie.

!\[\[12\_Ransom\_image004.png]]

This is clearly base64, and we can deocde it.

&#x20;

!\[\[12\_Ransom\_image005.png]]

&#x20;

There seems to be this weird value and also another iv here. This looks like an AES encryption, and they give us the MAC, the IV and the ciphertext.

This does not seem decryptable, so let's move on for now.

&#x20;

Let's take another look at the request.

When submitting the request with no other parameters, it gives us this JSON format.

!\[\[12\_Ransom\_image006.png]]

&#x20;

!\[\[12\_Ransom\_image007.png]]

This is in JSON, and if they process in JSON, the request may also be able to process that.

&#x20;

!\[\[12\_Ransom\_image008.png]]

&#x20;

Cool, now we have this.

We could brute force the password, but that isn't the way.

&#x20;

Tried to do some SQL injection related stuff, but didn't work. I then tried setting it as true, and I was able to get into the box.

!\[\[12\_Ransom\_image009.png]]

&#x20;

With this, we can logon.

&#x20;

!\[\[12\_Ransom\_image0010.png]]

We can go ahead and grab the user flag and also the homedirectory.zip.

{width="5.78125in" height="1.0in"}

This has another password to it. Zip2john this.

&#x20;

It seems that within the file, there is the id\_rsa folder.

!\[\[12\_Ransom\_image0012.png]]

We need to get that so we can SSH in as whatever user this is.

&#x20;

Seems that all of these files here have some form of encryption, and it's not possible to break into this.

!\[\[12\_Ransom\_image0013.png]]

&#x20;

From here, it seems that we need to read more about the information in the files, otherwise we would have no chance of getting stuff.

{width="11.197916666666666in" height="4.989583333333333in"}

This one seems to point us towards bash\_logout.

&#x20;

I remember a long time ago, the CRC there was able to be brute forced and decrypt the file should it be small enough for us to brute force out the content.

The Cyclic Redundancy Check is used just to check whether or not the contents of the zip file are correct.

&#x20;

After lots of googling and checking around for zip file decryption, I stumbled across this when looking for zip file encryption flaws:

!\[\[12\_Ransom\_image0015.png]]

&#x20;

This seems to take the a file with the same CRC with that tool. This works using the known plaintext attack, whereby two zip files have the same CRC value and hence text within them. This attack only works on zipCrypto and legacy decryption algorithms.

&#x20;

Interesting.

&#x20;

!\[\[12\_Ransom\_image0016.png]]

Taking a look at this, we can just use our own bash\_logout to create another zip file with the same CRC value.

!\[\[12\_Ransom\_image0017.png]]

From this, we have created another zip file with the exact same CRC value. This is because both the files have the same text within it that does not change often.

{width="8.03125in" height="2.0729166666666665in"}

This works out, and we get some keys.

&#x20;

{width="9.1875in" height="1.3333333333333333in"}

&#x20;

{width="4.770833333333333in" height="2.9375in"}

Decrypted all of it!

&#x20;

Read the public key to gain the username:

!\[\[12\_Ransom\_image0021.png]]

&#x20;

Now SSH in as this user.

!\[\[12\_Ransom\_image0022.png]]

&#x20;

When running ID, it seems we are in both sudo and LXD.

!\[\[12\_Ransom\_image0023.png]]

&#x20;

However, we do not have any passwords to do anything with this.

&#x20;

For now, we will just run a linpeas to enumerate for me. I was particularly interested in the web page stuff. I wanted to see the authentication mechanisms and the correct password for that user.

!\[\[12\_Ransom\_image0024.png]]

But, there's no Mysql installed on this server.

&#x20;

Seems that the webroot is here:

!\[\[12\_Ransom\_image0025.png]]

Within this file I found the AuthController:

!\[\[12\_Ransom\_image0026.png]]

This should contain the password for the Laravel stuff.

&#x20;

{width="7.364583333333333in" height="2.6354166666666665in"}

&#x20;

Well, that was simple.

&#x20;
