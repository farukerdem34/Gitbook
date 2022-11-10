# Scrambled

Scrambled

Monday, June 20, 2022

8:05 PM

Nmap scan:

!\[\[26\_Scrambled\_image001.png]]

&#x20;

This might be an AD related machine.

&#x20;

Port 80:

!\[\[26\_Scrambled\_image002.png]]

&#x20;

When looking at IT services, we get this notification.

!\[\[26\_Scrambled\_image003.png]]

&#x20;

So there's no usage of NTLM within the network.

Page Source:

!\[\[26\_Scrambled\_image004.png]]

&#x20;

This page has other pages within it, and we can request for a password reset.

&#x20;

SupportRequest:

!\[\[26\_Scrambled\_image005.png]]

&#x20;

There's this domain here, and also a user called ksimpson.

We can add the domain to our hosts file.

!\[\[26\_Scrambled\_image006.png]]

&#x20;

NewUser:

!\[\[26\_Scrambled\_image007.png]]

&#x20;

There are 4 different departments, and also one special access called the sales app access, which presumably is just part of the network somewhere.

&#x20;

SalesOrders:

!\[\[26\_Scrambled\_image008.png]]

&#x20;

There's this other domain here, and we now know the dc is called dc1.scrm.local, which we can add to our hosts file again. Interestingly, this logs in to port 4411.

&#x20;

Port 4411:

{width="3.6979166666666665in" height="0.875in"}

&#x20;

PasswordReset:

!\[\[26\_Scrambled\_image0010.png]]

&#x20;

There's just this one thing, and we need to leave some details with the IT support line.

&#x20;

Apart from all of this, there are no other indications of null sessions of anything.

&#x20;

DNS:

Trying zone transfers and to find out more about the domain reveals this.

!\[\[26\_Scrambled\_image0011.png]]

&#x20;

Not too significant.

&#x20;

Web Server Exploits:

There are no other directories, so I presume the entry is using one of the four pages that we have found earlier.

&#x20;

Creation of new user:

!\[\[26\_Scrambled\_image0012.png]]

This does not really do anything.

&#x20;

Let's map out what we know:

* There is a user called ksimpson, with an email presumably like ksimpson@scrm.local or ksimpson@scramblecorp.com
* There is a chance to submit some form of password reset, but there are no mail ports open on this machine.
* There is a service running on port 4411, and there are some commands that can be inputted there.
* There are no null sessions that are exploitable.
* NTLM hashes are disabled, meaning that the main way of authentication is either password or tickets.

&#x20;

There is a need to reset the password somehow.

&#x20;

Brute Force of Users:

{width="11.5625in" height="0.5625in"}

&#x20;

This reveals some useless information.

&#x20;

Randomly trying getTGT:

{width="6.21875in" height="1.53125in"}

&#x20;

The password was ksimpson...which was weird considering that we had to guess this portion.

Oh well.

&#x20;

Passing the ticket:

{width="4.21875in" height="0.6354166666666666in"}

&#x20;

{width="6.5625in" height="2.8854166666666665in"}

&#x20;

Now we had some access to the shares with this one ticket, but we need to think what else could it be used for authentication at.

Converting to hashcat:

{width="5.875in" height="1.0in"}

This gave me nothing.

&#x20;

Anyways, after experimenting with the suite of impacket tools and -k, I found this worked.

{width="6.75in" height="1.40625in"}

&#x20;

!\[\[26\_Scrambled\_image0019.png]]

&#x20;

!\[\[26\_Scrambled\_image0020.png]]

&#x20;

There's this PDF here.

{width="5.09375in" height="0.5729166666666666in"}

&#x20;

PDF:

!\[\[26\_Scrambled\_image0022.png]]

&#x20;

Interesting, there was an attacker being able to retrieve credentials from an SQL Database...Meaning we should proceed with SQL instead. Take note there's no NTLM stuff within this, so we need to purely use Kerberos.

&#x20;

Kerberoasting MSSQL:

{width="9.822916666666666in" height="2.7916666666666665in"}

&#x20;

Interesting, we can find these SPNs here.

&#x20;

Getting tickets:

{width="9.75in" height="3.6041666666666665in"}

&#x20;

Unfortunately, GetUserSPN.py is stupid.

Using python2 works (Idk why!).

&#x20;

{width="9.46875in" height="0.78125in"}

&#x20;

!\[\[26\_Scrambled\_image0026.png]]

&#x20;

We can then proceed to crack this.

{width="9.59375in" height="2.2916666666666665in"}

&#x20;

Now we have more credentials and another user to play with.

Remember, there's no NTLM stuff, so we need to convert this password to a ticket somehow.

&#x20;

We can first convert this to an NTLM hash.

!\[\[26\_Scrambled\_image0028.png]]

&#x20;

So for this, we would need to craft a silver ticket to authenticate. (I tried using it with sqlsvc with the usage of getTGT, but it does not let me authenticate for some reason)

&#x20;

First we need to find out the domain SID of this SQLSvc user.

{width="8.791666666666666in" height="4.270833333333333in"}

&#x20;

Found it.

Then, we can use impacket-ticketer to continue on.

{width="9.84375in" height="1.3333333333333333in"}

&#x20;

!\[\[26\_Scrambled\_image0031.png]]

&#x20;

Now that we are here, we can just enable xp\_cmdshell as we always do.

!\[\[26\_Scrambled\_image0032.png]]

&#x20;

We now finally have RCE on the machine.

&#x20;

Shell:

!\[\[26\_Scrambled\_image0033.png]]

&#x20;

{width="6.833333333333333in" height="2.1979166666666665in"}

&#x20;

PE1:

There are other users on this machine.

!\[\[26\_Scrambled\_image0035.png]]

&#x20;

!\[\[26\_Scrambled\_image0036.png]]

&#x20;

There is SeImpersonatePrivilege enabled, however, printspoofer does not work. I suspect this has been patched or there are just no printers enabled on the device.

&#x20;

When looking at the systeminfo, this does not appear vulnerable to potatoes as well.

!\[\[26\_Scrambled\_image0037.png]]

&#x20;

As the SQL user, we should have some checking over the SQL related databases.

We can try to enumerate the database.

!\[\[26\_Scrambled\_image0038.png]]

&#x20;

!\[\[26\_Scrambled\_image0039.png]]

&#x20;

We can enumerate this user's database.

!\[\[26\_Scrambled\_image0040.png]]

&#x20;

We can find a password!

Now, we cannot evil-winrm in.

&#x20;

This is because the user is not part of the Remote Management Group.

However, we can remote powershell this.

!\[\[26\_Scrambled\_image0041.png]]

&#x20;

Shell:

!\[\[26\_Scrambled\_image0042.png]]

&#x20;

{width="6.583333333333333in" height="2.1979166666666665in"}

&#x20;

Flag:

!\[\[26\_Scrambled\_image0044.png]]

0c6991576ae656de162b1c00a5c1f1ed\


PE2:

Now that we have access to this user, we can begin to look around for privilege escalation vectors.

We can now access all of the shares that we were blocked off until now.

&#x20;

When in the shares directory, do dir /s to see all files.

!\[\[26\_Scrambled\_image0045.png]]

&#x20;

We can see the ScrambleClient.exe that we found earlier.

There's also this DLL that we need to somehow use as well.

&#x20;

We can transfer this back to our machine for easy analysis.

&#x20;

!\[\[26\_Scrambled\_image0046.png]]

&#x20;

{width="3.0416666666666665in" height="0.5104166666666666in"}

&#x20;

We have to somehow use this in order to do stuff in the machine.

When using strings on this, we can find this little part here.

!\[\[26\_Scrambled\_image0048.png]]

&#x20;

There is this deserialise from base64 portion. This would mean that there is some form of deserialisation exploit going on here, because taking user input and doing that is never a good thing.

&#x20;

Within this there also lies some stuff regarding the commands.

!\[\[26\_Scrambled\_image0049.png]]

&#x20;

When connecting to the port, we can see this.

{width="5.40625in" height="1.875in"}

&#x20;

There is some form of base64 stuff being outputted.

&#x20;

I'm assuming that upload\_order would take this and then try to do something with it.

I used revshells.com to try and generate a working shell.

!\[\[26\_Scrambled\_image0051.png]]

&#x20;

When trying to upload this, we get this error.

!\[\[26\_Scrambled\_image0052.png]]

&#x20;

It does tell us the starting bit, but I have no indication to when it ends. Since there's some form of serialisation going on, we can just use ysoserial.

However, since this is a Windows machine, we would need to use ysoserial.exe instead of the linux version.

&#x20;

We can download this on our Windows VM and start it.

!\[\[26\_Scrambled\_image0053.png]]

&#x20;

This generates something very similar to what we saw earlier.

&#x20;

Shell:

!\[\[26\_Scrambled\_image0054.png]]

&#x20;

{width="5.25in" height="1.59375in"}

&#x20;

{width="6.614583333333333in" height="2.2291666666666665in"}

&#x20;

Flag:

!\[\[26\_Scrambled\_image0057.png]]

E1ffbc0b41543d6927d6ec47d479c267

&#x20;

Fuck this box.

NTLM hashes and tickets are so confusing.

&#x20;

Things I've learnt (which is a lot):

* That tickets can be stolen from ridiculous places
* Kerberoasting can happen anywhere, even when we don't even have a shell. Sometimes the order can be scrambled. (ffs)
* Enumerating databases was ok.
* Ysoserial.exe exists and binary formatting for windows exists. Jesus. This took so long to research

&#x20;

I did resort to a few hints regarding the serialisation.

&#x20;

Turns out there are other methods. We could have used RottenPotato or something to find CLSIDs to work with, that's how others got it so fast.

&#x20;

&#x20;
