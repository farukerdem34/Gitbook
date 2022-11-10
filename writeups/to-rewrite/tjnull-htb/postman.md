# Postman

Postman

Wednesday, 9 February 2022

8:47 pm

This is a Linux box,

Target IP is 10.10.10.160.

My IP is 10.10.16.5.

&#x20;

!\[\[31\_Postman\_image001.png]]

That's pretty cool, there's a redis server.

&#x20;

So Redis is apparently a database, and acts as an in-memory data structure store. It's mainly used for real-time analytics.

&#x20;

Visited the port 80 web server, encountered this.

&#x20;

!\[\[31\_Postman\_image002.png]]

&#x20;

So this is just something to look at really.

Directory enumeation next.

&#x20;

!\[\[31\_Postman\_image003.png]]

&#x20;

There is a login page on the webmin server.

&#x20;

!\[\[31\_Postman\_image004.png]]

I do not have any credentials yet.

&#x20;

Let's try looking at the Redis database that is present on the machine.

Found the tools on hacktricks, and began with using redis-cli in order to gain a connection. Which it accepted despite having no credentials.

!\[\[31\_Postman\_image005.png]]

&#x20;

From there, we can start to look around.

&#x20;

!\[\[31\_Postman\_image006.png]]

&#x20;

Good to know the version. Apparently connection to Redis would mean that there is a potential RCE that can be used here.

!\[\[31\_Postman\_image007.png]]

&#x20;

This is for a webshell, but we could also use SSH, through gaining a private key or something.

&#x20;

This would involve creating a key pair on our machine, and transferring that into the authorized\_keys file inside the Redis server, allowing us to SSH over!

&#x20;

{width="7.6875in" height="9.166666666666666in"}

&#x20;

Do these steps to launch the key file into the Redis server.

&#x20;

!\[\[31\_Postman\_image009.png]]

&#x20;

And take these to save.

From there, we can SSH directly into postman!

&#x20;

!\[\[31\_Postman\_image0010.png]]

&#x20;

We spawn in as Redis, and we have no access to many things. I took a look around and found this id\_rsa.bak file inside the /opt directory.

!\[\[31\_Postman\_image0011.png]]

I printed the output and copied that over to my machine.

&#x20;

Before we crack SSH keys, we need to feed it to SSH2John first.

&#x20;

!\[\[31\_Postman\_image0012.png]]

Afterwards we can crack it.

&#x20;

{width="9.697916666666666in" height="2.875in"}

So this is the password.

&#x20;

From the SSH shell we have, simply change user.

&#x20;

!\[\[31\_Postman\_image0014.png]]

Grab the user flag.

&#x20;

Ported over LinEnum.sh.

&#x20;

Anyways, with Matt, it did not reveal much. I forgot I had a Webmin server!

&#x20;

Using Matt:computer2008, I was able to log in to that.

!\[\[31\_Postman\_image0015.png]]

&#x20;

Found this page that works with the 1.910 version of Webmin installed here.

Lots of vulnerabilities available.

&#x20;

Ran this [script](https://github.com/NaveenNguyen/Webmin-1.910-Package-Updates-RCE/blob/master/exploit\_poc.py) I found.

{width="14.854166666666666in" height="2.8125in"}

&#x20;

Listener port grants us with root shell.

&#x20;

!\[\[31\_Postman\_image0017.png]]

&#x20;

Rooted.

&#x20;

This was a fun box, taught me a lot about Redis and stuff. From the title I expected to pentest SMTP or POP3.

&#x20;

&#x20;
