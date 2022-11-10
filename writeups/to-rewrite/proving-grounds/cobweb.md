# Cobweb

Cobweb

Sunday, July 3, 2022

9:45 PM

Nmap scan:

!\[\[71\_Cobweb\_image001.png]]

&#x20;

FTP:

!\[\[71\_Cobweb\_image002.png]]

&#x20;

We can get these logs and then view some stuff.

&#x20;

!\[\[71\_Cobweb\_image003.png]]

&#x20;

This page would produce some form of SQL query.

!\[\[71\_Cobweb\_image004.png]]

&#x20;

Then we can see an eval function, which is always good to see (exploit wise). We can see that this is basically the function of the initial port 80.

&#x20;

The other log files don't tell us much as of now.

{width="9.697916666666666in" height="3.4166666666666665in"}

&#x20;

So we need to exploit this eval function.

We can view this code clearer within Burp.

!\[\[71\_Cobweb\_image006.png]]

&#x20;

So basically, we can see that it selects route\_string from somewhere to go...somewhere.

WIP.

So we have this page that would display the whole php code, and I'm guessing this is for the main page.

In this case, we can see that the web page itself is vulnerable to SQL Injection because the stuff

&#x20;

Code analysis:

{width="8.635416666666666in" height="4.8125in"}

&#x20;

This would have a REDIRECT\_URL parameter that is being set, and if it is set, basically it would pass something to the function. Else it would redirect us back to the home directory of /. What's weird is that it calls the eval() function to do the function returning.

&#x20;

!\[\[71\_Cobweb\_image008.png]]

&#x20;

The function is simple, but basically it seems that it would set the route\_string to something. It seems that this database is the one that is being used for the returning of web pages instead of a traditional thing.

&#x20;

Basically, this would mean that paramater of index.php is already vulnerable.

Logically, this would mean the route\_string would have to be set as index.php, because there's no other page within the machine.

&#x20;

So in this case, we can append some SQL injection queries to the end of index.php.

!\[\[71\_Cobweb\_image009.png]]

&#x20;

We have achieved some form of SQL Injection.

&#x20;

Now we just need to input some type of shell.

We know that this database would basically be the one that has the webpages, so we should make an insert function into the thing.

We can test further to get it to dump out the phpinfo();

!\[\[71\_Cobweb\_image0010.png]]

&#x20;

!\[\[71\_Cobweb\_image0011.png]]

&#x20;

Then we can see the webroot. It's at /var/www/html or /usr/share/httpd. We can also see how the REDIRECT\_URL can eb controlled by us kind of.

&#x20;

So we know the query is something like

&#x20;

Select page from webpage where route\_string = \\"\<string here>\\";

To escape this, we would just need to append a query.

" as first character to escape it.

&#x20;

We can either append to the database, or we can put a cmd.php within the file directory.

Else, we can also read the database config.php and then connect through there somehow.

&#x20;

I was able to do this weird thing.

!\[\[71\_Cobweb\_image0012.png]]

&#x20;

I'm not sure what's happening here, but it seems that I was able to pass a legitimate query into the website, and it sems that we are unable to do UNION Injection for whatever reason.

&#x20;

We can try INSERT INTO now.

!\[\[71\_Cobweb\_image0013.png]]

&#x20;

Something like this should theoretically work.

I tried the ping command, and it worked.

!\[\[71\_Cobweb\_image0014.png]]

&#x20;

{width="9.760416666666666in" height="2.0416666666666665in"}

&#x20;

Now we just need to gain a reverse shell.

!\[\[71\_Cobweb\_image0016.png]]

&#x20;

!\[\[71\_Cobweb\_image0017.png]]

&#x20;

{width="8.229166666666666in" height="2.3125in"}

&#x20;

We can get the config.php now.

!\[\[71\_Cobweb\_image0019.png]]

&#x20;

&#x20;

Flag:

!\[\[71\_Cobweb\_image0020.png]]

&#x20;

&#x20;

PE:

SUID binaries:

!\[\[71\_Cobweb\_image0021.png]]

&#x20;

!\[\[71\_Cobweb\_image0022.png]]

&#x20;

At shell doesn't work too well.

&#x20;

Screen:

!\[\[71\_Cobweb\_image0023.png]]

&#x20;

We can exploit screen.

[https://www.exploit-db.com/exploits/41154](https://www.exploit-db.com/exploits/41154)

&#x20;

However, there are a few edits that need to be made.

Used walkthroguh as I was not sure on how to do this.

&#x20;

!\[\[71\_Cobweb\_image0024.png]]

&#x20;

So basically, we need to check if the directory itself can have setsuid binaries within our code, and then we would need to modify our exploits.

&#x20;

We can use /var/lib/php/session and then exploit it accordingly.

{width="9.291666666666666in" height="3.40625in"}

&#x20;

Follow the exploit given.

&#x20;

Flag:

!\[\[71\_Cobweb\_image0026.png]]

&#x20;
