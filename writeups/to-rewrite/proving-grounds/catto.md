# Catto

Catto

Monday, 28 March 2022

10:10 am

A Linux machine with these ports.

&#x20;

!\[\[40\_Catto \_image001.png]]

&#x20;

!\[\[40\_Catto \_image002.png]]

&#x20;

Port 8080 reveals this website.

!\[\[40\_Catto \_image003.png]]

Within the page source, we can see this:

!\[\[40\_Catto \_image004.png]]

&#x20;

There seems to be some form of email input area, with a few departments including a /dev/null option, which is interesting.

&#x20;

The form method seems to take a few different parameters. Trying to send a few POST requests doesn't really do anything.

&#x20;

Port 30330 reveals this:

!\[\[40\_Catto \_image005.png]]

&#x20;

Port 33047 reveals this

!\[\[40\_Catto \_image006.png]]

&#x20;

This is interesting, as there are some APIs to fuzz from here.

!\[\[40\_Catto \_image007.png]]

Sending a post request reveals an error

&#x20;

Sending some JSON objects reveals an error.

!\[\[40\_Catto \_image008.png]]

Now we at least know the user is marcus.\


While looking through the requests for the blog server hosted by gatsby, I found this:

!\[\[40\_Catto \_image009.png]]

This was the result of a GET /\_\_services, and this gave us more about the web server and its uses.

&#x20;

When looking again at the website, it has TypeScript enabled.

&#x20;

!\[\[40\_Catto \_image0010.png]]

&#x20;

So basically, there are methods of which we can actually execute stuff.

This web browser also has a graphiQL directory, which allows us to execute some GraphiQL code, and from this we may be able to get a password.

!\[\[40\_Catto \_image0011.png]]

It was found here when looking at gatsbyjs telemetry server code, which seems to execute certain requests based on the events and errors. The best bet here is the trackEvents link.

&#x20;

When looking at the graphql page, we seem to be able to query some hidden directories.

!\[\[40\_Catto \_image0012.png]]

&#x20;

This can be done using the allSitePage here.

&#x20;

!\[\[40\_Catto \_image0013.png]]

We are able to get our one web page which does not normally exist on the book reviews page.

&#x20;

!\[\[40\_Catto \_image0014.png]]

And now we have a password.

&#x20;

Now, we need to find some users.

This gave us a hint towards finding a Minecraft user.

!\[\[40\_Catto \_image0015.png]]

&#x20;

And there are some users here! Knowing that the user is marcus, we can try to SSH in.

&#x20;

Using this, we can SSH in as marcus.

!\[\[40\_Catto \_image0016.png]]

&#x20;

Within the home directory, there's this little .config file here.

&#x20;

!\[\[40\_Catto \_image0017.png]]

There's also a .bash file there.

&#x20;

!\[\[40\_Catto \_image0018.png]]

&#x20;

Within this there's a password.

!\[\[40\_Catto \_image0019.png]]

Translated, this means nothing however.

&#x20;

There must be some form of hidden key or something, since this is not base64 encoded.

Within the files, there are other base64 binaries.

&#x20;

!\[\[40\_Catto \_image0020.png]]

I have never seen base64key, so let's try it.

&#x20;

{width="6.71875in" height="1.0520833333333333in"}

I'm not entirely sure what this one is, but it seems to encrypt and decrypt things accordingly.

We need some form of PRIVATEKEY, which we can use marcus's password and attempt this.

&#x20;

{width="8.760416666666666in" height="0.5416666666666666in"}

This gives us this password.

&#x20;

{width="8.4375in" height="5.895833333333333in"}

&#x20;

&#x20;
