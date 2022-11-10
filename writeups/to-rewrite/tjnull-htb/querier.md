# Querier

Querier

Sunday, 13 March 2022

3:00 pm

Another windows machine.

The target IP is 10.10.10.125.

My IP is 10.10.16.9.

&#x20;

!\[\[83\_Querier\_image001.png]]

&#x20;

!\[\[83\_Querier\_image002.png]]

&#x20;

Enum4linux returned nothing of use, but smbclient does.

&#x20;

!\[\[83\_Querier\_image003.png]]

&#x20;

There's one disk that's interesting, and we can connect to it.

!\[\[83\_Querier\_image004.png]]

There's this file here. Let's grab it.

&#x20;

Well, this is a microsoft excel file. Opening it using LibreOffice does nothing, as this has some macros to it.

Googling a little bit, I found [oletools](https://github.com/decalage2/oletools), which is a tool that can be used to analyse Microsoft files.

&#x20;

!\[\[83\_Querier\_image005.png]]

There's some VBA macros here.

&#x20;

There's some SQL code here.

&#x20;

!\[\[83\_Querier\_image006.png]]

&#x20;

!\[\[83\_Querier\_image007.png]]

Grab that password.

The username seems to be 'reporting'

&#x20;

Let's log in:

!\[\[83\_Querier\_image008.png]]

Looking at all of the databases, we have a few:

!\[\[83\_Querier\_image009.png]]

&#x20;

The databases don't look very interesting.

I ventured to HackTricks to determine more things about what can be done with MS SQL.

!\[\[83\_Querier\_image0010.png]]

This looks rather interesting. Let's try it.

&#x20;

Start an SMB server.

&#x20;

From there, we can pick up a hash from the logs.

!\[\[83\_Querier\_image0011.png]]

&#x20;

Ran a john on this and got a password.

{width="6.90625in" height="1.59375in"}

Right.

&#x20;

Now, let's try to evilwin-rm in.

Unsurprisingly, this does not work in our favour.

&#x20;

Let's try logging in again to the SQL server with this user.

&#x20;

!\[\[83\_Querier\_image0013.png]]

&#x20;

From here, [this](https://rioasmara.com/2020/01/31/mssql-rce-and-reverse-shell-xp\_cmdshell/) blog post shows that this user has elevated privileges, and we are able to execute commands here. Following his commands, I managed to achieve RCE:

&#x20;

!\[\[83\_Querier\_image0014.png]]

Great. Now let's see if I can download or even ping stuff.

!\[\[83\_Querier\_image0015.png]]

&#x20;

{width="5.5625in" height="1.96875in"}

&#x20;

We can get our nc onto this machine, start a listener port and just watch!

!\[\[83\_Querier\_image0017.png]]

&#x20;

!\[\[83\_Querier\_image0018.png]]

Right.

&#x20;

We have a lot of privileges as this user.

!\[\[83\_Querier\_image0019.png]]

Intriguing.

&#x20;

Anyways, systeminfo reveals that this has quite a few hotfixes, so I don't think I'll be exploiting this.

Ran a winpeas on this just to do it for me, a bit lazy now.

&#x20;

Loads of vulnerabilities.

!\[\[83\_Querier\_image0020.png]]

&#x20;

Right, so let's view some of them

&#x20;

Weirdly enough, I found these:

{width="6.625in" height="2.5729166666666665in"}

Well....

Evil-winrm and we are root.

!\[\[83\_Querier\_image0022.png]]

&#x20;
