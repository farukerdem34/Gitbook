# XposedAPI

XposedAPI

Thursday, 17 March 2022

8:13 pm

A Linux machine with the following ports:

!\[\[18\_XposedAPI\_image001.png]]

&#x20;

Lots of filtered ports, the firewall seems strong with this one.

!\[\[18\_XposedAPI\_image002.png]]

Gunicorn is an API, so let's head to the web server and see how we can fuzz it.

&#x20;

!\[\[18\_XposedAPI\_image003.png]]

&#x20;

Well, we have this.

I ran a directory enumeration too.

&#x20;

When getting the version, we get this:

!\[\[18\_XposedAPI\_image004.png]]

&#x20;

It seems that the application also has an update, and let's try to get it to reach our python servers.

I switched to Burpsuite as this might be easier.

When trying to get an update, we're hit with the invalid username:

!\[\[18\_XposedAPI\_image005.png]]

When trying to restart, we get this:

!\[\[18\_XposedAPI\_image006.png]]

&#x20;

!\[\[18\_XposedAPI\_image007.png]]

&#x20;

This sends another HTTP request somewhere else.

!\[\[18\_XposedAPI\_image008.png]]

&#x20;

This is the result of the website.

&#x20;

Anyways, to spoof past this WAF, we need to make it look like it came from us. We should do some SSRF, which would involve appending the header X-Forwarded-For header and 127.0.0.1. This would make the website think the request is coming from itself.

&#x20;

!\[\[18\_XposedAPI\_image009.png]]

&#x20;

From here, we can determine that we have LFI.

&#x20;

!\[\[18\_XposedAPI\_image0010.png]]

&#x20;

!\[\[18\_XposedAPI\_image0011.png]]

&#x20;

There's one user there, with the name clumsyadmin.

We can now get the username and URL stuff that we need.

&#x20;

!\[\[18\_XposedAPI\_image0012.png]]

&#x20;

!\[\[18\_XposedAPI\_image0013.png]]

So, now we can literally take shells from our device and load it onto the device.

&#x20;

!\[\[18\_XposedAPI\_image0014.png]]

I had some trouble getting a shell that worked.

&#x20;

I tried chaining multiple comamnds together like this:

!\[\[18\_XposedAPI\_image0015.png]]

This worked in executing my payload.

&#x20;

!\[\[18\_XposedAPI\_image0016.png]]

&#x20;

!\[\[18\_XposedAPI\_image0017.png]]

&#x20;

!\[\[18\_XposedAPI\_image0018.png]]

Interestingly, it seems that we are using wget here.

&#x20;

Well, we can use this to read the /etc/shadow file, and overwrite it with our own malicious file.

&#x20;

All we need to do is get it.

!\[\[18\_XposedAPI\_image0019.png]]

&#x20;

!\[\[18\_XposedAPI\_image0020.png]]

&#x20;

The hashes are not crackable, so I appended the /etc/passwd file instead.

!\[\[18\_XposedAPI\_image0021.png]]

Generate a quick password, this one was 'hello'.

&#x20;

Get the /etc/passwd file using the post-file method.

&#x20;

!\[\[18\_XposedAPI\_image0022.png]]

Append this new user.

Use wget to save it and overwrite the current /etc/passwd file.

!\[\[18\_XposedAPI\_image0023.png]]

&#x20;

!\[\[18\_XposedAPI\_image0024.png]]

&#x20;

&#x20;
