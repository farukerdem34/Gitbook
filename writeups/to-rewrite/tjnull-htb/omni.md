# Omni

Omni

Monday, 21 February 2022

9:29 pm

&#x20;

This is a Windows machine.

My IP is 10.10.16.9.

The target IP 10.10.10.204.

&#x20;

Enumeration first! Always determine all the possible ports that are open before going further.

!\[\[50\_Omni\_image001.png]]

There are these ports open.

&#x20;

I did a further nmap scan to see what version and systems are running. Concurrently, I investigated the web port.

Turns out this website requires a password to enter.

&#x20;

!\[\[50\_Omni\_image002.png]]

&#x20;

!\[\[50\_Omni\_image003.png]]

&#x20;

So there are these running on the machine.

Let's take a look at every single port that we can.

&#x20;

I took a look at port 8080 in-depth, as this gave me a login page to tent do.

&#x20;

!\[\[50\_Omni\_image004.png]]

This time however, this gave me a Windows Device Portal of some sort. I have never seen that before.

A bit of Google-fu led to me that this is some kind of Windows IoT device, and some articles that it can be exploited.

&#x20;

Some more Googling led me to this software called [SirepRAT](https://github.com/SafeBreach-Labs/SirepRAT), which is basically using a remote access trojan to do our bidding. This might not be a boring old Windows machine after all.

&#x20;

This is basically something that allows for RCE. I tested it out, and it seems to return valid inputs.

!\[\[50\_Omni\_image005.png]]

&#x20;

We seem to be able to get files and also upload files. Let's try uploading some form of nc.exe and then from there, executing it to give us a shell.

&#x20;

So the right way to execute stuff is through the use of cmd.exe, and then calling powershell to execute code for us.

&#x20;

!\[\[50\_Omni\_image006.png]]

&#x20;

From here, let's try to get nc64.exe onto the machine.

&#x20;

!\[\[50\_Omni\_image007.png]]

In this case, spool was used because it typically is something that can be exploited.

&#x20;

Now, Do this command to initiate nc.

!\[\[50\_Omni\_image008.png]]

We should get a shell.

!\[\[50\_Omni\_image009.png]]

&#x20;

Cool!

Now we are in. We should go find out more about this machine.

I looked around, and it seems that we aren't even close to capturing the flags yet.

&#x20;

Within the main \ directory, there was a Data file, something I have not seen before. Taking a look at this revealed that there was an app Account and Administrator account.

!\[\[50\_Omni\_image0010.png]]

Within these files, there was a user.txt and root.txt, but these were...different.

!\[\[50\_Omni\_image0011.png]]

&#x20;

!\[\[50\_Omni\_image0012.png]]

&#x20;

There was some kind of funny password, which was encrypted or something.

There was also an iot-admin.xml file, which contained credentials for an omni administrator and a password.

!\[\[50\_Omni\_image0013.png]]

I didn't know where to start with this one, and I knew there was a better way of doing this but I did not know how...

Hence, I resorted to a walkthrough and found out there was a simple hidden credential file that would save us the time.

&#x20;

!\[\[50\_Omni\_image0014.png]]

&#x20;

This contained the username and password.

&#x20;

Having this, I tried logging into the website again.

&#x20;

Logged in as admin!

!\[\[50\_Omni\_image0015.png]]

Within this, there was a command runner, which acted as an RCE.

&#x20;

!\[\[50\_Omni\_image0016.png]]

&#x20;

What I tried was to use the same netcat and connect back to my machine, this time with a cmd.exe shell.

I assumed so because there was one single file, called hardening.txt that I not allowed to view. Also I wanted to get out of powershell.

!\[\[50\_Omni\_image0017.png]]

&#x20;

!\[\[50\_Omni\_image0018.png]]

This would let me do more stuff, hopefully.

&#x20;

Now, from here, let's view that file.

&#x20;

!\[\[50\_Omni\_image0019.png]]

Interesting, so this would limit the firewall and also the default administrator password. It seems that we need to decrypt this thing no matter what huh.

&#x20;

Anyways looking back at the encrypted file. Let's start with the IoT-admin file.

!\[\[50\_Omni\_image0013.png]]

This is a management automation Pscredential output file or something.

&#x20;

We need to use this function called GetNetworkCredential() or something, based on Microsoft Documentation.

&#x20;

!\[\[50\_Omni\_image0020.png]]

A lot of Googling later, this would involve PowerShell, and that the shell we got was kind of useless.

&#x20;

Anyways, this is the output of the command to get the flags.

!\[\[50\_Omni\_image0021.png]]

Do the same for user.txt, and this should give us the flag we want.

&#x20;

This box was a bit tricky for me, and I relied on online walkthroughs.

&#x20;

&#x20;
