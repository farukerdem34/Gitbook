# Atom

Atom

Monday, 14 March 2022

12:54 pm

This is a Windows machine.

The target IP is 10.10.10.237.

My IP is 10.10.17.233.

&#x20;

!\[\[84\_Atom\_image001.png]]

Nmap version scan is a bit weird on this server.

&#x20;

!\[\[84\_Atom\_image002.png]]

Port 445 and a Redis server again.

&#x20;

&#x20;

Website, which is the same for both HTTP and HTTPS:

!\[\[84\_Atom\_image003.png]]

&#x20;

Enum4linux reveals nothing interesting.

SMB client does however.

!\[\[84\_Atom\_image004.png]]

&#x20;

!\[\[84\_Atom\_image005.png]]

Gonna get all of these files within my system.

&#x20;

The connection to this box is a little unstable, as it stops quite often.

&#x20;

The PDF was about installing the application to our machine.

!\[\[84\_Atom\_image006.png]]

&#x20;

Based on this, it seems that we have to place some application within the client folders to check if updates have been applied correctly.

!\[\[84\_Atom\_image007.png]]

&#x20;

Perhaps we have to install their application to get us what we need.

Let's try installing this application.

&#x20;

!\[\[84\_Atom\_image008.png]]

Has to be an exe file it seems.

&#x20;

When we click it, we will get a zip file that contains one exe file.

&#x20;

Now, let's try to disassemble this .exe file somehow.

&#x20;

!\[\[84\_Atom\_image009.png]]

&#x20;

Seems that when opening the Exe file, we get these.

!\[\[84\_Atom\_image0010.png]]

&#x20;

There's a plugin file, which looks to cintain all the different DLLs and stuff.

!\[\[84\_Atom\_image0011.png]]

&#x20;

The app-64.7z file is something that looks interesting, and when extracted, reveals more information about the different types of stuff.

&#x20;

!\[\[84\_Atom\_image0012.png]]

&#x20;

There was one file that stood out:

!\[\[84\_Atom\_image0013.png]]

This is something called electron.

&#x20;

So Electron is a free and open-source software framework developed by Github, which would allow for the development of desktop GUI applications using web technologies. It seems that this thing was running on an electron application framework, and to update it we just need to place this folder within the client files.

&#x20;

I searched for electron app exploits, which led to me to [this](https://blog.yeswehack.com/yeswerhackers/exploitation/pentesting-electron-applications/) blog installing the tool called asar. There are asar files within this that can be reversed engineered, of which HTML, JS and CSS files are all put together.

&#x20;

Sure enough, within the resources file there was an app.asar file.

!\[\[84\_Atom\_image0014.png]]

From here, we can take the files out.

&#x20;

!\[\[84\_Atom\_image0015.png]]

&#x20;

There was one main.js file.

!\[\[84\_Atom\_image0016.png]]

These were the libraries installed.

&#x20;

I googled more about electron exploits, and seeing the electron-updater library, followed by the method of which we can update the app, which is to place it within the SMB folders, seems like the updater is the vulnerable one.

&#x20;

Found [this](https://blog.doyensec.com/2020/02/24/electron-updater-update-signature-bypass.html) for electron-updater exploits, which leads to RCEs.

&#x20;

Also found this:

!\[\[84\_Atom\_image0017.png]]

&#x20;

In which case, we do have control over the update server.

Looking at the JSON file within the electron folder, we can identify the version is indeed vulnerable.

!\[\[84\_Atom\_image0018.png]]

&#x20;

From here, the same blog as above highlighted we can do this:

!\[\[84\_Atom\_image0019.png]]

Through appending a single quote, we can basically execute exe files from there after placing them both within the SMB server.

&#x20;

There was also this portion:

!\[\[84\_Atom\_image0020.png]]

So during a software update, it will check on this file and attempt to do something about that.

Seems like we have our exploit here.

&#x20;

Let's attempt it.

&#x20;

So first, create a malicious reverse shell exe file, and rename it such that it is like so:

&#x20;

{width="9.791666666666666in" height="1.8645833333333333in"}

&#x20;

!\[\[84\_Atom\_image0022.png]]

Then take the shasum based on the command given to me by the blog.

&#x20;

!\[\[84\_Atom\_image0023.png]]

&#x20;

Now, we need to create a latest.yml file that would execute this for us.

&#x20;

!\[\[84\_Atom\_image0024.png]]

This should work out, and I just changed the release date to today's date.

&#x20;

Now, connect to the SMB client and just place these files within the directory.

!\[\[84\_Atom\_image0025.png]]

Waited around for a bit.

Didn't work the first time, but my files were 'consumed'.

Tried again.

&#x20;

I then realised that my IP was wrong, supposed to be 10.10.16.9. Oops.

&#x20;

Created another rev shell:

!\[\[84\_Atom\_image0026.png]]

Also, I experiemented with this using URLs, and it also works.

!\[\[84\_Atom\_image0027.png]]

We can vary the URL parameter of the YML file.

&#x20;

Tried again:

!\[\[84\_Atom\_image0028.png]]

&#x20;

!\[\[84\_Atom\_image0029.png]]

This time we need to set up one HTTP server and we can see it happening.

&#x20;

!\[\[84\_Atom\_image0030.png]]

&#x20;

!\[\[84\_Atom\_image0031.png]]

And there we go.

&#x20;

Grab the user flag, and let's explore around here.

Within downloads, I saw this file.

&#x20;

!\[\[84\_Atom\_image0032.png]]

A quick google search reveals that this file actually stores encrypted passwords or something.

&#x20;

!\[\[84\_Atom\_image0033.png]]

&#x20;

Real convenient.

Within it, there was a .cfg file.

From here, we can get the encrypted password.

!\[\[84\_Atom\_image0034.png]]

&#x20;

Looking at that exploit, we can easily decrypt this.

&#x20;

!\[\[84\_Atom\_image0035.png]]

&#x20;

!\[\[84\_Atom\_image0036.png]]

Cool!

&#x20;

Now, let's try an evil-winrm in.

&#x20;

Except that isn't the right password.

This is the redis server password, but anyways still valid!

&#x20;

!\[\[84\_Atom\_image0037.png]]

Great.

Now, let's look around this database.

!\[\[84\_Atom\_image0038.png]]

We have these keys, and there's one user key there.

&#x20;

!\[\[84\_Atom\_image0039.png]]

Another encrypted password.

&#x20;

Same algorithm though!

!\[\[84\_Atom\_image0040.png]]

&#x20;

!\[\[84\_Atom\_image0041.png]]

Ez.

&#x20;

&#x20;

&#x20;
