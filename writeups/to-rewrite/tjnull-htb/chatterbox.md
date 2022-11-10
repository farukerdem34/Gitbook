# Chatterbox

Chatterbox

Wednesday, 9 February 2022

4:32 pm

This is a Windows machine.

My IP is 10.10.16.5.

The target IP is 10.10.10.74.

&#x20;

Something to do with powershell, so I prepped nishang in case.

An initial scans reveals that the first 1000 ports are not open. Hence append the -p- flag to our nmap in order to start our recon.

!\[\[30\_Chatterbox\_image001.png]]

&#x20;

In the end, there were two ports, port 9255 and port 9256 that were open.

!\[\[30\_Chatterbox\_image002.png]]

&#x20;

Port 9255 was of little interest, whereas port 9256 were vulnerable to buffer overflows.

{width="12.53125in" height="1.28125in"}

&#x20;

Seems that the service Achat was running on the application, and perhaps we can use this somehow.

!\[\[30\_Chatterbox\_image004.png]]

When looking at the script, it seems to suggest a MSFvenom command that would generate a reverse shell on the machine.

&#x20;

!\[\[30\_Chatterbox\_image005.png]]

&#x20;

It seems that all this payload does is open up calc.exe on the Windows machine, nothing else. So let's try to insert a reverse shell there and make it happen.

&#x20;

&#x20;

!\[\[30\_Chatterbox\_image006.png]]

&#x20;

Used this to generate a whole lot of characters that we would use to replace the current payload within the exploit code. Then I copy and pasted it into the exploit script. Also we need to reset the machine, because if we use it the application should crash and unless it has a recurring script to keep it open (which I don't think it has), we would be unable to exploit anything.

&#x20;

After resetting the box, we can run the script and make sure to have a listener port open in case of anything.

Just like that we should gain a shell.

&#x20;

!\[\[30\_Chatterbox\_image007.png]]

&#x20;

Problem is, it is not sustained at all. It disconnects after a while. I tried with a different payload. Also turns out we don't need to keep resetting the machine, great. Tried with -p windows/shell\_reverse\_tcp instead, understanding that this would force a connection.

&#x20;

!\[\[30\_Chatterbox\_image008.png]]

Cool, we're in.

We spawned in as the user.

&#x20;

Let's check the systeminfo to see what is this thing running.

{width="9.40625in" height="8.4375in"}

&#x20;

I see hotfixes, no potatoes should work on this thing.

I quickly ported over winPEAS using certutil to see what we can do with this.

&#x20;

There are a few things we can do to privilege escalate, some of which involve PowerShell.

&#x20;

More interestingly, I saw this.

&#x20;

{width="11.875in" height="1.1875in"}

So Alfred can access the Admin account.

&#x20;

Additionally I saw the scheduled applications, which makes everyone's life easier.

&#x20;

!\[\[30\_Chatterbox\_image0011.png]]

&#x20;

This was running some reset file, likely keeping Achat alive.

!\[\[30\_Chatterbox\_image0012.png]]

Cool!

So this means that we basically own the Admin desktop. Definitely some sort of misconfiguration present there.

&#x20;

However, accessing the root.txt is not allowed.

!\[\[30\_Chatterbox\_image0013.png]]

&#x20;

Hmm, but Alfred owns the desktop, so how can we be denied of permissions?

I wanted to check the permission on this file, and stackoverflow provided a good answer (as always).

&#x20;

There was a command called icacls.

Which is basically meaning I see Access Control Lists, something I needed.

!\[\[30\_Chatterbox\_image0014.png]]

So turns out the root.txt belongs to the Admin

!\[\[30\_Chatterbox\_image0015.png]]

&#x20;

So the flags mean different things, OI means object inherit, and files will inherit its permissions, CI means container inherit, and IO is inherit only. We can see that the Root.txt file is just belonging to the Admin. The :F means that the user just owns this File as well.

&#x20;

Since we own the Desktop, we can reassign permissions I think.

&#x20;

Reading the documentation of icacls, there seems to be a /grant flag that we can use.

{width="9.197916666666666in" height="1.5520833333333333in"}

This would mean that we can reassign the text file to others. This works out well, and we can reassign the file to the user alfred's as well.

&#x20;

!\[\[30\_Chatterbox\_image0017.png]]

&#x20;

An interesting box.
