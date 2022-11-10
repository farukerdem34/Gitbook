# Nest

Nest

Monday, 14 March 2022

8:55 pm

This is a Windows machine.

The target IP is 10.10.10.178.

My IP is 10.10.16.9.

!\[\[87\_Nest\_image001.png]]

Interesting!

Ran a more detailed scan and enum4linux just to see if I could get anything.

&#x20;

Enum4linux did return something.

!\[\[87\_Nest\_image002.png]]

&#x20;

From here, let's go see the stuff about the SMB.

Within the Data disk, there were a few directories, and I recursively got them all.

&#x20;

Detailed scan of Nmap seems to reveal that there is literally only port 445 to care about. Port 4386 is unassigned.

Users had directories, of which I had no access to them.

&#x20;

!\[\[87\_Nest\_image003.png]]

&#x20;

The other directories pretty much had nothing for me.

Within the data directory, there were two .txt files.

&#x20;

!\[\[87\_Nest\_image004.png]]

&#x20;

!\[\[87\_Nest\_image005.png]]

The welcome email contains some form of credentials.

&#x20;

!\[\[87\_Nest\_image006.png]]

Maintenance work is useless.

&#x20;

But anyway now we have some creds, let's smb once more.

Within the directory of this Tempuser, there was one text file.

&#x20;

!\[\[87\_Nest\_image007.png]]

This was empty however...

&#x20;

{width="9.96875in" height="2.7604166666666665in"}

There were still other files to go through!

&#x20;

Turns out this had a load of folders.

&#x20;

!\[\[87\_Nest\_image009.png]]

&#x20;

Within the Microsoft Options.xml file, there was a hint here.

&#x20;

!\[\[87\_Nest\_image0010.png]]

This could be something I have to use later.

&#x20;

The config.xml file had a hint here.

!\[\[87\_Nest\_image0011.png]]

It says the file IT has the file Carl within it.

&#x20;

Sure enough, I can access it.

&#x20;

!\[\[87\_Nest\_image0012.png]]

&#x20;

From here, I can see there are VB projects, of which when getting all of them, we have more things to look through.

!\[\[87\_Nest\_image0013.png]]

&#x20;

From here, the file basically shows us an encryption thing, whereby we have to play around with this string.

&#x20;

The funny thing is, the script itself contains the Decrypt function, which indicates we are supposed to compile it and stuff.

Well, not to worry, someone has already done that [online.](https://dotnetfiddle.net/bjoBP6)

I do not want to waste time installing the Visual Studio stuff needed, compiling and then looking through variable data.

&#x20;

Now, we just need to find that one string to decrypt.

!\[\[87\_Nest\_image0014.png]]

Now we have the file name, but where is it?

!\[\[87\_Nest\_image0015.png]]

There it is.

!\[\[87\_Nest\_image0016.png]]

There's the hash.

When decrypted online, I found the password.

!\[\[87\_Nest\_image0017.png]]

&#x20;

{width="9.697916666666666in" height="2.7291666666666665in"}

Great, now we have access to C.Smith.

Grab the user flag and let's press on.

&#x20;

!\[\[87\_Nest\_image0019.png]]

There's this new directory here.

&#x20;

Within it, there's 2 config files and one .exe file. Oh dear.

&#x20;

!\[\[87\_Nest\_image0020.png]]

&#x20;

Within the config file we saw earlier, there was mention of port 4386, the unknown port we worked with earlier.

&#x20;

Turns out, there port is running that exe file and we probably have to do something to it to get our root password.

&#x20;

We can look around this and see that there's an LDAP folder.

&#x20;

{width="4.21875in" height="3.1666666666666665in"}

To use any of the services however, we need a password.

&#x20;

!\[\[87\_Nest\_image0022.png]]

This file should have it but it's empty.

Perhaps there are alternate data streams.

&#x20;

{width="7.177083333333333in" height="2.09375in"}

Was rught!

&#x20;

!\[\[87\_Nest\_image0024.png]]

Cool.

&#x20;

!\[\[87\_Nest\_image0025.png]]

Cool, now we have more information.

&#x20;

From here, we are able to read the administrator hash.

&#x20;

!\[\[87\_Nest\_image0026.png]]

Now, I'm assuming that we have to disassemble that .exe file.

&#x20;

Right, we need to get a .NET decompiler on this, seeing that the author likes to use these kinds of files.

&#x20;

Get it from here: [https://github.com/icsharpcode/AvaloniaILSpy/releases/download/v7.1-rc/Linux.x64.Release.zip](https://github.com/icsharpcode/AvaloniaILSpy/releases/download/v7.1-rc/Linux.x64.Release.zip)

Decompiling the application, we can see this portion of code:

&#x20;

!\[\[87\_Nest\_image0027.png]]

This seems to use some strings, so let's just try this with the earlier code.

!\[\[87\_Nest\_image0028.png]]

Well.

&#x20;

!\[\[87\_Nest\_image0029.png]]

&#x20;

!\[\[87\_Nest\_image0030.png]]

Another trick!

&#x20;

Wait...

{width="10.041666666666666in" height="2.6354166666666665in"}

I have the admin password, I can just go into C$...

&#x20;

Grab the root flag.

&#x20;

&#x20;

&#x20;
