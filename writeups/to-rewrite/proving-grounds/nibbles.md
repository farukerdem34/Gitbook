# Nibbles

Nibbles

Wednesday, 16 March 2022

12:01 am

The target IP is 192.168.73.47.

My IP is 192.168.49.73.

&#x20;

Looks to be a rather fun box.

!\[\[05\_Nibbles\_image001.png]]

Detailed nmap scan time, and normally I try anonymous logins with FTP servers.

Doesn't work.

&#x20;

!\[\[05\_Nibbles\_image002.png]]

&#x20;

Port 80:

!\[\[05\_Nibbles\_image003.png]]

Not too sure what this website means.

&#x20;

The link to another page leads to this page2.html page.

&#x20;

!\[\[05\_Nibbles\_image004.png]]

The Internet Cats link leads to what is called nightmare .jpg.

&#x20;

!\[\[05\_Nibbles\_image005.png]]

I ran a gobuster on this website, with the html extension enabled. Revealed nothing. Seems like the port 80 is useless.

I decided to focus on port 21, because that sems to be the most potential for any files to be enumerated.

&#x20;

On HackTricks, I found this list of FTP default credentials to try, and why not.

Created one user name list and one password list from this, and ran hydra.

&#x20;

!\[\[05\_Nibbles\_image006.png]]

Left it for a while.

This didn't work.

&#x20;

Last option is to enumerate that mysql server.

!\[\[05\_Nibbles\_image007.png]]

Some default credentials are postgres:postgres.

&#x20;

Now, from here we can see that we are some form of root.

Funnily enough, we can read the hash from the postgres user.

&#x20;

!\[\[05\_Nibbles\_image008.png]]

&#x20;

!\[\[05\_Nibbles\_image009.png]]

Now, let's try an SSH.

Does not work though.

&#x20;

We can actually read files from this method:

!\[\[05\_Nibbles\_image0010.png]]

From here, now I know we can execute code on this thing.

&#x20;

!\[\[05\_Nibbles\_image0011.png]]

&#x20;

Bash shells don't work, so let's try netcat.

Oddly, no shells I tried were working, but ping works so definitely this is the way:

{width="9.979166666666666in" height="1.65625in"}

&#x20;

I tried creating a simple bash shell, and then execute it through the same manner.

!\[\[05\_Nibbles\_image0013.png]]

&#x20;

!\[\[05\_Nibbles\_image0014.png]]

&#x20;

!\[\[05\_Nibbles\_image0015.png]]

I swear these machines hate me.

&#x20;

Was convinced that by now, some shells should be working.

So many boxes from PG just don't work for some reason...I was doing this exploit exactly as needed.

Perhaps my linux machine is not letting the machines.

&#x20;

Decided to go with the web kali instead since it seems PG does not function well for me at night. Could be because the USA is awake.

We already know what to do with this command, we just need to execute it easily and then from there PE.

&#x20;

Left with little choice, and trying again to no avail, I used MSF.

!\[\[05\_Nibbles\_image0016.png]]

If this does not work, then PG hates me.

Tried with a different port.

&#x20;

!\[\[05\_Nibbles\_image0017.png]]

Seems port 21 works...

!\[\[05\_Nibbles\_image0018.png]]

&#x20;

Seems that I was using a port the firewall was blocking. (I did try 80 and 443, but it still blocked me).

Ran a quick check on linpeas and the stuff we can execute.

!\[\[05\_Nibbles\_image0019.png]]

From here, we can just do this from GTFObins.

!\[\[05\_Nibbles\_image0020.png]]

&#x20;

!\[\[05\_Nibbles\_image0021.png]]

From this, we can priv escalate.

&#x20;

!\[\[05\_Nibbles\_image0022.png]]

Dones.

&#x20;

Not too sure why my reverse shells weren't working...will have to avoid using MSF.

So far, I miss HTB.

&#x20;

&#x20;
