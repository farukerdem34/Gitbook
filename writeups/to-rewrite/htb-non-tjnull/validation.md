# Validation

Validation

Monday, 28 March 2022

4:37 pm

A Linux machine with these ports:

&#x20;

!\[\[07\_Validation\_image001.png]]

Gonna ignore the filtered ports and focus on the rest.

!\[\[07\_Validation\_image002.png]]

&#x20;

Port 80 just has this:

&#x20;

!\[\[07\_Validation\_image003.png]]

Interesting.

&#x20;

When we enter our credentials and stuff, we just get this.

&#x20;

!\[\[07\_Validation\_image004.png]]

When looking at requests within burp, the usage of account.php has loads of SQL related queries.

!\[\[07\_Validation\_image005.png]]

&#x20;

&#x20;

!\[\[07\_Validation\_image006.png]]

We can also figure out the database from this, and how it processes the data from the user.

!\[\[07\_Validation\_image007.png]]

&#x20;

When appending stuff to the country, we get this error.

&#x20;

It seems that there is a failure for the SQL query. Since the cookie used is a hash, I can sort of construct the query to be

**SELECT \[hash] FROM \[country] --**

&#x20;

The attacking point here would be the country parameter.

&#x20;

!\[\[07\_Validation\_image008.png]]

This payload eventually worked as I was able to get out something.

&#x20;

We are in the registration database.

!\[\[07\_Validation\_image009.png]]

&#x20;

From here, we have to be able to identify the other tables

!\[\[07\_Validation\_image0010.png]]

&#x20;

&#x20;

This seems to be the registration table, but I want to know the other tables.

I knew there would be some kind of table called users within this database, so I did this query next.

!\[\[07\_Validation\_image0011.png]]

Well, enough of this. Time to write in my own shell into the webserver.

&#x20;

!\[\[07\_Validation\_image0012.png]]

From this we can access our web shell through the web server.

!\[\[07\_Validation\_image0013.png]]

Cool.

&#x20;

!\[\[07\_Validation\_image0014.png]]

&#x20;

!\[\[07\_Validation\_image0015.png]]

There's a config.php file with the database credentials within it, which I will look at a little later.

&#x20;

!\[\[07\_Validation\_image0016.png]]

&#x20;

From here, we can grab the user flag through the user called htb.

!\[\[07\_Validation\_image0017.png]]

That password was the root password after all.

&#x20;

Simple box.

&#x20;

&#x20;
