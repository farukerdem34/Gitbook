# Devoops

Devoops

Sunday, 6 March 2022

12:03 pm

This is a Linux machine.

My IP is 10.10.16.2.

The target IP is 10.10.10.91.

&#x20;

Let's enumerate this first.

There's only one port it appears.

!\[\[68\_Devoops\_image001.png]]

&#x20;

!\[\[68\_Devoops\_image002.png]]

&#x20;

Well. No exploits on searchsploit.

Website:

!\[\[68\_Devoops\_image003.png]]

Not sure what this means for us, but let's do a directory enumeration,

Let's also monitor the web traffic and see what gets generated when we reload the page and so on.

&#x20;

When we refresh the page, it seems to request for two directories.

!\[\[68\_Devoops\_image004.png]]

Reloaded it twice.

&#x20;

It seems to draw the image there from /feed directory.

Directory scan:

!\[\[68\_Devoops\_image005.png]]

&#x20;

There's an upload folder, hmm. We already know the /feed folder would lead us to the image shown.

&#x20;

!\[\[68\_Devoops\_image006.png]]

Seems to only allow for XML documents, which might lead to an XXE injection vulnerability.

Let's try to upload some picture to see what happens.

&#x20;

!\[\[68\_Devoops\_image007.png]]

It takes it somewhere...

&#x20;

!\[\[68\_Devoops\_image008.png]]

This is the POST request being submitted. Let's try some PHP files and try to find out where these go.

&#x20;

When I tried to upload some form of XML document, this is what I got.

!\[\[68\_Devoops\_image009.png]]

&#x20;

There was a hint given to us however, using the XML elements it gave us:

!\[\[68\_Devoops\_image0010.png]]

Perhaps we should upload some kind of XML File with these located within it.

&#x20;

!\[\[68\_Devoops\_image0011.png]]

Tried stuff like this and did not get a code 200 back.

&#x20;

I googled around and determined it could be an entry instead.

So I changed it to this and it managed to return an output.

!\[\[68\_Devoops\_image0012.png]]

This gave me a file path, as well as the possible uploads that this could go to. We also know that the user is called roosa.

&#x20;

I began to think, would it be possible to inject some code in somehow?

Interestingly, I was able to cause an error when referring to an object without defining it.

!\[\[68\_Devoops\_image0013.png]]

&#x20;

Would it be possible to leak out SSH keys or execute a reverse shell into this?

&#x20;

Needed to find a way to inject the definition DOCTYPE.

!\[\[68\_Devoops\_image0014.png]]

This works. Now, let's change that to execute some code.

&#x20;

!\[\[68\_Devoops\_image0015.png]]

&#x20;

!\[\[68\_Devoops\_image0016.png]]

Well, now let's look around the home directory of roosa.

&#x20;

!\[\[68\_Devoops\_image0017.png]]

Cool!

&#x20;

!\[\[68\_Devoops\_image0018.png]]

&#x20;

!\[\[68\_Devoops\_image0019.png]]

Just like that we're in.

&#x20;

From here, time to PE.

&#x20;

!\[\[68\_Devoops\_image0020.png]]

We are part of the sudo group!

&#x20;

I still ran a linpeas in case.

!\[\[68\_Devoops\_image0021.png]]

&#x20;

When looking in the home directory, I found a .gitconfig file.

!\[\[68\_Devoops\_image0022.png]]

&#x20;

{width="3.7291666666666665in" height="1.0in"}

Always good to know.

&#x20;

So this means there's some kind of .git file somewhere.

&#x20;

!\[\[68\_Devoops\_image0024.png]]

There's one there.

Looking at the git logs, we can see this one,

&#x20;

!\[\[68\_Devoops\_image0025.png]]

&#x20;

!\[\[68\_Devoops\_image0026.png]]

Use git log -p to view everything.

&#x20;

!\[\[68\_Devoops\_image0027.png]]

Rooted.

Pretty easy box for us.

&#x20;

&#x20;
