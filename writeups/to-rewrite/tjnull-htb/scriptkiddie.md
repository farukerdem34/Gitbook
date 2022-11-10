# ScriptKiddie

ScriptKiddie

Tuesday, 22 February 2022

11:20 am

This is a Linux machine.

The target IP is 10.10.10.226.

My IP is 10.10.16.9

&#x20;

Looks to be a good box.

Enumerate this thing.

Nmap scan reveals the following:

!\[\[51\_ScriptKiddie\_image001.png]]

Let's take a look at this web page.

!\[\[51\_ScriptKiddie\_image002.png]]

Now this is really interesting, it seems to be a web page and it can scan IP addresses.

!\[\[51\_ScriptKiddie\_image003.png]]

&#x20;

There seems to be an ability to generate MSFvenom payloads as well, with multiple different compatible OS.

&#x20;

Searchsploit is also present on the machine.

Most interestingly, there's this upload button, wonder if we can make any funny payloads using this.

&#x20;

It allows us to upload a template file.

I tried generating a file, and this produced a payload that was valid for 5 minutes.

&#x20;

!\[\[51\_ScriptKiddie\_image004.png]]

&#x20;

Seems that when trying to upload a template file, this is the error I get based on what I uploaded.

&#x20;

!\[\[51\_ScriptKiddie\_image005.png]]

We need to get the correct file for the correct payload.

&#x20;

What if we were to use msfvenom to generate a payload to input into this? I generated an elf file and analysed the upload process in Burp.

{width="9.572916666666666in" height="2.0416666666666665in"}

&#x20;

!\[\[51\_ScriptKiddie\_image007.png]]

I opened up a listener port too, just in case.

&#x20;

In this case, it just produced this.

!\[\[51\_ScriptKiddie\_image008.png]]

Interesting, now I'm wondering what else we can possibly upload to get an error.

I can tell this is somehow executing code within the machine that is hosting this machine, so we need to try to get it to execute some form of shell I'll be uploading.

I took note that this was generating meterpreter shells, so I change my msfvenom shell to a meterpreter one.

This did not yield anything. I then googled meterpreter exploits, and this showed up.

&#x20;

!\[\[51\_ScriptKiddie\_image009.png]]

So there's a literal flaw in the MSFvenom version here. I used searchsploit for this.

!\[\[51\_ScriptKiddie\_image0010.png]]

There's literally only one, let's try that shall we?

&#x20;

!\[\[51\_ScriptKiddie\_image0011.png]]

I used a different script, seeing that the original did not work as well.

I changed the payload to a reverse shell there.

!\[\[51\_ScriptKiddie\_image0012.png]]

For this exploit, we would need to install apktool, jarsigner, and keytool on our machines.

Get it, then use the script.

&#x20;

{width="10.708333333333334in" height="3.1666666666666665in"}

&#x20;

The template has been generated.

&#x20;

!\[\[51\_ScriptKiddie\_image0014.png]]

Then just wait for it to generate something.

&#x20;

This did not work for me, and I resorted to using the MSF framework to create a payload instead.

{width="10.604166666666666in" height="7.78125in"}

Inject this payload into the machine, and we get a shell!

!\[\[51\_ScriptKiddie\_image0016.png]]

Through this, we can see that there's an app.py.

!\[\[51\_ScriptKiddie\_image0017.png]]

This is the application of which it generates the payloads and stuff.

&#x20;

Grab the user flag, and continue!

Looking at the /etc/passwd file, it seems that there are other users.

&#x20;

!\[\[51\_ScriptKiddie\_image0018.png]]

There's a pwn account.

Anyways, we can see that in this app.py script, there is a writing of code into this other file.

{width="9.84375in" height="2.0416666666666665in"}

&#x20;

!\[\[51\_ScriptKiddie\_image0020.png]]

&#x20;

Hmm. This seems to write to this empty file though, not sure if I need this.

Within the user pwn, there are two files.

!\[\[51\_ScriptKiddie\_image0021.png]]

I cannot check out recon, so let's check out this script instead.

{width="8.0625in" height="2.7291666666666665in"}

This seems to echo some stuff into the log, and then write more code within it.

Seems that it checks whether the log in hackers is being written to, and then it executes an nmap scan based on the code given to the hackers log file.

&#x20;

The format of the code written to that of hackers is datetime.datetime.now() and then some IP address I think, which equates to something like

**"\[2021-05-28 12:37:32.655374] 10.10.16.9"**

&#x20;

I echoed in some code to see what would happen.

&#x20;

This is what happened.

{width="13.114583333333334in" height="3.5520833333333335in"}

&#x20;

So this thing executes some form of code, can we perhaps append a bash script into it for another reverse shell?

&#x20;

This is what I attempted. I echoed in some other script.

This gave me a shell!

&#x20;

!\[\[51\_ScriptKiddie\_image0024.png]]

&#x20;

!\[\[51\_ScriptKiddie\_image0025.png]]

So the last character there is to comment the rest of the script. This is to maintain our shell.

Now we need to do into the recon folder, which we weren't given access to earlier.

&#x20;

There is just the logs of the recons here. Which is not too interesting.

Well, since we now own this script, I could certainly simply add my SSH key into the authorized keys of root and SSH in.

&#x20;

!\[\[51\_ScriptKiddie\_image0026.png]]

Does not work however...

!\[\[51\_ScriptKiddie\_image0027.png]]

&#x20;

Damn.

I copied over linpeas.sh to check on what I can really do, plus I was a bit lazy to recon myself.

I tried running sudo -l, which paid off.

!\[\[51\_ScriptKiddie\_image0028.png]]

&#x20;

Seems that we can run msfconsole...

This is great because, we can just run commands from msfconsole as root.

I can just run sudo msfconsole, and this is basically a root shell.

Pwned!

!\[\[51\_ScriptKiddie\_image0029.png]]

&#x20;

{width="6.4375in" height="4.708333333333333in"}

&#x20;
