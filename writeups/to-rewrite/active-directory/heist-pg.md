# Heist (PG)

Heist (PG)

Tuesday, 22 March 2022

12:56 pm

Windows machine with these ports:

!\[\[12\_Heist (PG)\_image001.png]]

We start with an enum4linux, to begin enumerating a bit on the SMB side.

All forms of SMB do not work here, now we can move to enumerating the ldap ports with nmap and ldapsearch.

&#x20;

!\[\[12\_Heist (PG)\_image002.png]]

Grab the domains and stuff.

&#x20;

Ldap Nmap returns good information, so let's try to get an ldapsearch going to find out more about this port.

Ldap also does not return much.

&#x20;

Anyways, let's take a look at port 8080, which seems to have some form of web browser going for it.

D

!\[\[12\_Heist (PG)\_image003.png]]

There's just this, and the first thing to come to mind is to try an RFI.

&#x20;

!\[\[12\_Heist (PG)\_image004.png]]

&#x20;

!\[\[12\_Heist (PG)\_image005.png]]

Right, so we can indeed get files into the machine from here.

This web browser does not seem to execute things on our behalf, as it does not run on PHP or execute powershell stuff.

&#x20;

Interestingly, it can connect to the localhost as well.

Other than that, I could not get it to execute anything but simply print the page.

&#x20;

Now, I was wondering whether or DNS matters. It doesn't, nothing can be gotten from that.

&#x20;

I took a hint, and it says the website is vulnerable to SSRF?

&#x20;

Turns out, we can use Responder to poison the network, and from there we can grab a hash!

&#x20;

{width="9.791666666666666in" height="2.1979166666666665in"}

&#x20;

{width="7.34375in" height="2.3541666666666665in"}

We can crack this to get some form of hash. Now, the hint said not to use RDP, so let's try evil-winrm.

!\[\[12\_Heist (PG)\_image008.png]]

Great!

!\[\[12\_Heist (PG)\_image009.png]]

&#x20;

{width="5.864583333333333in" height="1.875in"}

There seems to be this to do list here. Interestingly, there's this file sent to admin and migrate to apache or something. Not sure what this is about.

&#x20;

!\[\[12\_Heist (PG)\_image0011.png]]

From here, we can view some stuff regarding enox.

!\[\[12\_Heist (PG)\_image0012.png]]

There seems to be a service account there, and that's probably where we're headed to next if possible.

&#x20;

!\[\[12\_Heist (PG)\_image0013.png]]

There's also this updater thingy, which is very interesting. We can probably hijack one of those exe or DLL files with our own to have persistence.

&#x20;

Anyways, I wanted to do a winpeas in case.

!\[\[12\_Heist (PG)\_image0014.png]]

Seems that lsass is running on this, and I could potentially get some stuff from it.

&#x20;

Now, most interesting to me is that service account for apache, I wonder what it can do.

Anyways, the most interesting privilege I have is this SeMachineAccountPrivilege.

&#x20;

!\[\[12\_Heist (PG)\_image0015.png]]

Not too sure why this has been enabled, but it appears I can add users to the domain.

&#x20;

Since we are indeed a web admin, we should have some form of control over the apache server somehow.

Lots of googling brought me here:

&#x20;

!\[\[12\_Heist (PG)\_image0016.png]]

&#x20;

!\[\[12\_Heist (PG)\_image0017.png]]

&#x20;

Did this, and it looks like I can view this user a bit more.

&#x20;

!\[\[12\_Heist (PG)\_image0018.png]]

Seems that our password is msDS protected, and we need to get it out somehow.

!\[\[12\_Heist (PG)\_image0019.png]]

&#x20;

!\[\[12\_Heist (PG)\_image0020.png]]

We get a long string of numbers.

Anyways there are tools like this [https://github.com/rvazarkar/GMSAPasswordReader](https://github.com/rvazarkar/GMSAPasswordReader) which allow for us to directly decrypt the password into hashes.

&#x20;

{width="11.260416666666666in" height="4.208333333333333in"}

We get some hashes, let's try to pass the hash.

&#x20;

Tried with the first hash, and got it to work!

!\[\[12\_Heist (PG)\_image0022.png]]

&#x20;

!\[\[12\_Heist (PG)\_image0023.png]]

We have a new privilege, which is SeRestorePrivilege.

&#x20;

!\[\[12\_Heist (PG)\_image0024.png]]

From github, we can do this it seems.

!\[\[12\_Heist (PG)\_image0025.png]]

Do that, and then we need RDP access to continue.

Once we lock the console there, we can access this:

!\[\[12\_Heist (PG)\_image0026.png]]

&#x20;

!\[\[12\_Heist (PG)\_image0027.png]]

Cool!

&#x20;

!\[\[12\_Heist (PG)\_image0028.png]]

That was a really cool exploit.

&#x20;

&#x20;
