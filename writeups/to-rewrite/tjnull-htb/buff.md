# Buff

Buff

Tuesday, 15 February 2022

5:00 pm

This is a Windows machine.

My IP is 10.10.16.9.

The target IP is 10.10.10.198.

&#x20;

Let's dive right into enumeration.

Based on the statistics alone, seems that the user shell of this box is relatively easier compared to the root shell. Windows machines are generally not my favourite because I like Linux. I need to learn PowerShell...

&#x20;

Interesting, seems that the ports are not open. Ran the scan again with a -Pn, which is a flag that would not ping the host in case firewalls were active.

&#x20;

!\[\[40\_Buff\_image001.png]]

&#x20;

Cool, got a port.

More interestingly, this is running Windows XP likely, and this is more of an interest to me. Perhaps there are a few exploits lying around just for this vulnerable version of Windows.

&#x20;

!\[\[40\_Buff\_image002.png]]

&#x20;

Anyways upon viewing the website, this is what we get.

!\[\[40\_Buff\_image003.png]]

&#x20;

Fitness related website. Interesting.

I ran a vuln.nse script scan, just to check on what else can be enumerated.

Additionally, I ran a directory enumeration to try and find out more.

&#x20;

!\[\[40\_Buff\_image004.png]]

There are a few directories we can choose to browse from.

&#x20;

I looked through some of them, and came across the contact.php.

&#x20;

!\[\[40\_Buff\_image005.png]]

Hmm, there's a software version there.

&#x20;

Searchsploit reveals that there are exploits for this one version of Gym Management Software.

!\[\[40\_Buff\_image006.png]]

&#x20;

Let's try that RCE.

&#x20;

Just like that I was in the machine already. Well, that was fast.

!\[\[40\_Buff\_image007.png]]

&#x20;

Grab the user flag while you're at it. This web shell is a little weird, because we are unable to even change directories. To grab the user flag, just use type and spell out the directory that leads to it.

&#x20;

!\[\[40\_Buff\_image008.png]]

Next, we should really change up this shell if we would like to properly connect. Looking at the systeminfo, I noted that this is a 64-bit PC.

!\[\[40\_Buff\_image009.png]]

Also it's Windows 10, something that is not so easy to exploit.

&#x20;

Anyways I knew that I had to transfer netcat 64-bit over to get a proper shell instead of this limited shell. Certutil and wget did not work, so I tried doing other methods.

&#x20;

Powershell seemed to work, but I have no idea where the file went.

Spent a while searching for that nc64.exe file.

While looking around, I found this exe file here.

!\[\[40\_Buff\_image0010.png]]

&#x20;

For now, let's keep in mind this is here, because I do not think it is something that is usually there.

!\[\[40\_Buff\_image0011.png]]

&#x20;

Right, so we're back here. With this box again.

&#x20;

Let's pick up where we left off.

Upload a web shell using smb.

!\[\[40\_Buff\_image0012.png]]

Then just run it and get a reverse shell.

&#x20;

Now time to check out that cloud thing.

IDA it.

&#x20;

{width="8.541666666666666in" height="4.09375in"}

Seems to something into a directory or something.

&#x20;

So turns out, there are vulnerabilities for this.

&#x20;

!\[\[40\_Buff\_image0014.png]]

Looking at the code, it seems that this is a simple BOF.

&#x20;

!\[\[40\_Buff\_image0015.png]]

This code here runs the calc.exe, but we want to gain a reverse shell or something.

&#x20;

That's easily changed. Next, there is the target host being 127.0.0.1. My

Looking at the ports that are listening, it seems that there is a port 8888.

&#x20;

!\[\[40\_Buff\_image0016.png]]

This should be the port of which the exe is listening on or something.

&#x20;

This would mean that we have to port forward this thing, and run the exploit from our machine.

Right, so I got chisel, which is a great tool for this.

&#x20;

Run chisel on our machine as the server.

!\[\[40\_Buff\_image0017.png]]

Get chisel on the other machine and run it as the client, this time connecting to port 1234 on our machine to its local port of 8888.

!\[\[40\_Buff\_image0018.png]]

Should work eventually.

&#x20;

Now that our ports are connected, we just need to run the exploit.

However, I tried for about an hour and it just didn't work.

&#x20;

Referred to walkthroughs, and I was doing exactly as needed. For now, this is how we clear the root. I don't know why my exploit isn't working, but I'm not going to apply the try harder mentality to this because I did exactly as needed.

&#x20;

&#x20;

&#x20;

&#x20;

&#x20;
