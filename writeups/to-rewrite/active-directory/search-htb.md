# Search (HTB)

Search (HTB)

Tuesday, June 28, 2022

5:43 PM

Nmap scan:

As per standard AD machine.

&#x20;

Port 80:

!\[\[20\_Search (HTB)\_image001.png]]

&#x20;

Some kind of company page again.

This time, there are some weird pictures.

!\[\[20\_Search (HTB)\_image002.png]]

&#x20;

See the picture saying Send Password to Hope Sharp and the password is as such? Weird...

&#x20;

Port 443:

It's the same website but with a certificate.

!\[\[20\_Search (HTB)\_image003.png]]

&#x20;

There is another domain here.

This does not aid us much now.

&#x20;

We have some form of credentials, so let's check it out using SMB.

&#x20;

Crackmapexec:

!\[\[20\_Search (HTB)\_image004.png]]

&#x20;

{width="9.822916666666666in" height="0.65625in"}

&#x20;

Cool.

We have access to some shares.

{width="9.78125in" height="3.6041666666666665in"}

&#x20;

Since we have credentials, why not bloodhound this?

&#x20;

Bloodhound:

{width="9.875in" height="0.9791666666666666in"}

&#x20;

We can find a few interesting things.

!\[\[20\_Search (HTB)\_image008.png]]

&#x20;

This guy is apparently some form of Domain Admin.

And there are 2 users that are kerberoastable.

!\[\[20\_Search (HTB)\_image009.png]]

&#x20;

Web\_svc looks interesting, and perhaps with our credentials we can quickly gain some form of credentials for this account.

{width="9.854166666666666in" height="2.4791666666666665in"}

&#x20;

From there, we can crack the hash.

{width="9.822916666666666in" height="2.6354166666666665in"}

&#x20;

So now we have access to the web\_svc user.

{width="9.822916666666666in" height="3.5in"}

&#x20;

&#x20;

This doesn't do much for us however, meaning we should probaby think of how to use this password and attempt some password spraying maybe,

Inside the RedirectedFolder share, there are lot of other usernames and stuff present.

!\[\[20\_Search (HTB)\_image0013.png]]

&#x20;

We can take a look at them and attempt some password spraying.

{width="9.8125in" height="0.7291666666666666in"}

&#x20;

This user has access to the helpdesk share.

{width="9.791666666666666in" height="3.6354166666666665in"}

&#x20;

The helpdesk share does not have anything, but perhaps there is stuff within the edgar.jacobs directory from SMB.

And indeed there is.

!\[\[20\_Search (HTB)\_image0016.png]]

&#x20;

There's a xlsx file here.

When downloaded and opened, we can see that there's this weird tab.

!\[\[20\_Search (HTB)\_image0017.png]]

&#x20;

And if we look closer, we realise that column C has been hidden.

WE can actually still unzip this and view the files within.

&#x20;

To bypass this, simply just make a copy of the excel file, rename it as a zip file and then unzip it.

{width="4.677083333333333in" height="1.90625in"}

&#x20;

Afterwards, go to the sheet we want, and remove the 'sheetProtection' tag from every single XML instance.

Then just compress and zip it back into an xlsx file.

&#x20;

Afterwards, we can read the ocntent within the machine.

!\[\[20\_Search (HTB)\_image0019.png]]

&#x20;

Interesting passwords indeed.

CME this.

{width="9.822916666666666in" height="4.34375in"}

&#x20;

We get a user and a password.

Within the redirectedFolder$, we can retrieve the user flag.

!\[\[20\_Search (HTB)\_image0021.png]]

&#x20;

Flag:

{width="3.9479166666666665in" height="0.9166666666666666in"}

&#x20;

PE:

Within the user's file, we can see that there are some .p12 and pfx files.

!\[\[20\_Search (HTB)\_image0023.png]]

&#x20;

We can download this and load it into our browsers.

&#x20;

It seems that both of these certificates require passwords however.

!\[\[20\_Search (HTB)\_image0024.png]]

&#x20;

We can pfx2john this easily.

{width="9.791666666666666in" height="3.8854166666666665in"}

&#x20;

The password for both files are the same, and we can now import them into my browser.

!\[\[20\_Search (HTB)\_image0026.png]]

&#x20;

When we try to visit the HTTPS page with /staff, we can get a prompt.

!\[\[20\_Search (HTB)\_image0027.png]]

Press Ok and we proceed to Powershell WA.

&#x20;

!\[\[20\_Search (HTB)\_image0028.png]]

We can login with sierra.frye and then the password we found earlier. For the computer name, exiftool on the documents does not do anything. I tested with search, research and other things, and research worked.

&#x20;

!\[\[20\_Search (HTB)\_image0029.png]]

&#x20;

When looking back at bloodhound, we can see that this user has certain privileges over the service accounts.

&#x20;

!\[\[20\_Search (HTB)\_image0030.png]]

&#x20;

We can see some form of thing here when looking at the privileges of sierra.frye.

So first thing we need to do is be able to execute commands over Bir-ADFS GMSA and then we would be able to reset the password of the Domain Admin basically.

&#x20;

!\[\[20\_Search (HTB)\_image0031.png]]

We can use gmsa.exe to read the NTLM hash.

&#x20;

Alternatively, we can just read the password.

!\[\[20\_Search (HTB)\_image0032.png]]

&#x20;

It looks like rubbish on purpose.

We can still just keep it within a variable that we control and can use to pass on the password to execute commands as another user.

!\[\[20\_Search (HTB)\_image0033.png]]

&#x20;

And we can gain RCE over the usage of this variable.

{width="6.6875in" height="1.8020833333333333in"}

&#x20;

Then we can reset the domain admin password.

!\[\[20\_Search (HTB)\_image0035.png]]

&#x20;

{width="9.854166666666666in" height="3.9166666666666665in"}

&#x20;

Shell:

{width="7.458333333333333in" height="2.7604166666666665in"}

&#x20;

Flag:

!\[\[20\_Search (HTB)\_image0038.png]]

&#x20;

That was a really enjoyable box with the certificates and stuff.

The hard part was finding out where to use that certificate.

&#x20;

&#x20;

&#x20;
