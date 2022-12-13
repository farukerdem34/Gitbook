# Access (AD)

Access (AD)

Saturday, July 2, 2022

12:01 PM

Nmap scan:

!\[\[111\_Access (AD)\_image001.png]]

&#x20;

Could be an AD machine.

&#x20;

Port 80:

A typical page for buying tickets.

Potential usernames:

!\[\[111\_Access (AD)\_image002.png]]

&#x20;

DNS is open, so we can try to find a domain.

&#x20;

LDAP:

!\[\[111\_Access (AD)\_image003.png]]

&#x20;

!\[\[111\_Access (AD)\_image004.png]]

&#x20;

Add this to our /etc/hosts file:

!\[\[111\_Access (AD)\_image005.png]]

&#x20;

Begin subdomain fuzzing and also DNS finding for another domain.

{width="9.791666666666666in" height="4.28125in"}

&#x20;

We can get a few more domains.

But these do nothing for us.

&#x20;

We can look at the web page a bit more and fuzz it.

!\[\[111\_Access (AD)\_image007.png]]

&#x20;

!\[\[111\_Access (AD)\_image008.png]]

&#x20;

I wfuzzed more directories.

!\[\[111\_Access (AD)\_image009.png]]

&#x20;

Nothing much.

&#x20;

File upload vulnerability:

When we try to buy a ticket, we can see that it allows us to upload an image actually.

!\[\[111\_Access (AD)\_image0010.png]]

&#x20;

I try to upload a webshell.

!\[\[111\_Access (AD)\_image0011.png]]

&#x20;

I get back this response.

!\[\[111\_Access (AD)\_image0012.png]]

&#x20;

That this file extension is not allowed. However, I think this can be bypassed.

!\[\[111\_Access (AD)\_image0013.png]]

&#x20;

I can send this in, and it would allow the uplaod.

However, we can see that even though it uploads, it appears that it does not execute code.

&#x20;

!\[\[111\_Access (AD)\_image0014.png]]

&#x20;

We can bypass this by including our own .htaccess file which would allow for php code to be executed within the /uploads directory.

[https://github.com/wireghoul/htshells](https://github.com/wireghoul/htshells)

&#x20;

This repo can be used to generate a htaccess shell for PHP and then execute code. This would modify the .htaccess file and allow for our code to be executed within the browser.

!\[\[111\_Access (AD)\_image0015.png]]

&#x20;

We send this as .htaccess, and then send a php cmd shell with a .gif extension.

&#x20;

!\[\[111\_Access (AD)\_image0016.png]]

&#x20;

!\[\[111\_Access (AD)\_image0017.png]]

&#x20;

RCE achieved.

&#x20;

Shell:

!\[\[111\_Access (AD)\_image0018.png]]

&#x20;

{width="6.84375in" height="2.1354166666666665in"}

Straightaway, we notice we are a service account.

&#x20;

!\[\[111\_Access (AD)\_image0020.png]]

&#x20;

There's another svc account here.

We can get powerview on here.

&#x20;

!\[\[111\_Access (AD)\_image0021.png]]

&#x20;

!\[\[111\_Access (AD)\_image0022.png]]

&#x20;

We can see that this has an SPN and we could potentially just kerberoast this user.

!\[\[111\_Access (AD)\_image0023.png]]

&#x20;

!\[\[111\_Access (AD)\_image0024.png]]

&#x20;

We can also use Kerberoast.ps1 to make it easier.

!\[\[111\_Access (AD)\_image0025.png]]

&#x20;

Transfer to our machine and crack it.

{width="9.760416666666666in" height="2.3854166666666665in"}

&#x20;

We now have a password to this user.

Then, we can bloodhound this.

&#x20;

{width="9.75in" height="2.96875in"}

&#x20;

Bloodhound doesn't reveal anything, which probably means that this is not ACL related or domain related.

Powershell remoting does not work either, and there's limited methods to gain a shell.

&#x20;

We can look at the shares and see what information is in there.

{width="9.979166666666666in" height="2.78125in"}

&#x20;

Hmm.

I attempted to use adPEAS. But nothing much came from it.

Anyways, there was a HTTPS port listening on the localhost.

!\[\[111\_Access (AD)\_image0029.png]]

So we cannot get a shell using this user, so there should be some way to find out how to exploit this without a shell.

&#x20;

Let's think of it this way, we have credentials, and we should be able to do something as this user.

There should be a binary that we can use to execute code as another user.

[https://github.com/antonioCoco/RunasCs](https://github.com/antonioCoco/RunasCs)

&#x20;

Using this binary, we can gain a shell as the new user.

!\[\[111\_Access (AD)\_image0030.png]]

&#x20;

!\[\[111\_Access (AD)\_image0031.png]]

&#x20;

Flag:

!\[\[111\_Access (AD)\_image0032.png]]

&#x20;

PE:

Within this, we have a unique privilege.

!\[\[111\_Access (AD)\_image0033.png]]

&#x20;

From this, we can basically make us be able to control the entire C:\ drive.

[https://github.com/CsEnox/SeManageVolumeExploit/releases](https://github.com/CsEnox/SeManageVolumeExploit/releases)

!\[\[111\_Access (AD)\_image0034.png]]

&#x20;

We can see that we have full control over the entire directory.

With this, we can gain a shell using wertrigger.

&#x20;

!\[\[111\_Access (AD)\_image0035.png]]

&#x20;

Then we can execute commands after running WerTrigger.

!\[\[111\_Access (AD)\_image0036.png]]

&#x20;

{width="7.34375in" height="2.2604166666666665in"}

&#x20;

Admin shelled.

&#x20;

Flag:

!\[\[111\_Access (AD)\_image0038.png]]

&#x20;
