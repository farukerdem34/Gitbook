# Shared

Shared

Thursday, July 28, 2022

12:08 PM

Nmap scan:

!\[\[37\_Shared\_image001.png]]

&#x20;

We would need to add shared.htb to /etc/hosts.

&#x20;

HTTPS site:

!\[\[37\_Shared\_image002.png]]

&#x20;

Pretty standard shop or corporate website.

&#x20;

Viewing the HTTPS cert, we can see that there are wildcards present.

!\[\[37\_Shared\_image003.png]]

&#x20;

We may need to do some subdomain fuzzing in order to find more websites.

&#x20;

When checking out the different products, there are some parameters passed to the URL.

!\[\[37\_Shared\_image004.png]]

&#x20;

We can test this for SQL injections or SSTI.

When viewing the requests, we can see that this is a PrestaShop thing.

!\[\[37\_Shared\_image005.png]]

&#x20;

PrestaShop does have vulnerabilities for it, and we just need to find out which version this is running.

&#x20;

Subdomain Fuzzing:

!\[\[37\_Shared\_image006.png]]

&#x20;

Checkout:

!\[\[37\_Shared\_image007.png]]

&#x20;

This looks quite vulnerable.

When we add items to our cart, we get this weird cookie called custom\_cart.

This can be tested for SQL Injection or something like that.

!\[\[37\_Shared\_image008.png]]

&#x20;

We can leverage this to gain some form of data from the database, because the custom cookie would basically cause the output to be printed on the checkout page.

!\[\[37\_Shared\_image009.png]]

&#x20;

So in this case, we can make use of some UNION injection.

Firstly, we can find out the database that is being used here.

!\[\[37\_Shared\_image0010.png]]

&#x20;

!\[\[37\_Shared\_image0011.png]]

&#x20;

So the database is checkout, and now we need to find a table.

&#x20;

The table name in this case is called user.

!\[\[37\_Shared\_image0012.png]]

&#x20;

Then we can find the username and password.

!\[\[37\_Shared\_image0013.png]]

&#x20;

!\[\[37\_Shared\_image0014.png]]

&#x20;

Since this can only display one item at once, we would need to enter two queries to find the password.

!\[\[37\_Shared\_image0015.png]]

&#x20;

This can be cracked.

!\[\[37\_Shared\_image0016.png]]

&#x20;

Then we can SSH in as this user.

!\[\[37\_Shared\_image0017.png]]

&#x20;

The flag would be located within the other user's directory, named dan\_smith.

!\[\[37\_Shared\_image0018.png]]

&#x20;

PE1:

We can get pspy into this server to enumerate what processes are being run as dan\_smith.

Interesting outputs:

!\[\[37\_Shared\_image0019.png]]

&#x20;

Root is running a redis server on the inside.

!\[\[37\_Shared\_image0020.png]]

&#x20;

The user with UID 1001, which is dan\_smith is running ipython.

We can find the version of ipython running, and it's 8.0.0

!\[\[37\_Shared\_image0021.png]]

&#x20;

This is vulnerable to an RCE exploit that should be the one used to gain access as the other user.

This one seems to be running some kind of script within the /opt/scripts\_review directory, which we have write access to.

&#x20;

In this case, we can go ahead and create a quick script that would let us read the private key of this Dan user.

We can copy the exploit based on this:

[https://github.com/ipython/ipython/security/advisories/GHSA-pq7m-3gw7-gq5x](https://github.com/ipython/ipython/security/advisories/GHSA-pq7m-3gw7-gq5x)

&#x20;

!\[\[37\_Shared\_image0022.png]]

&#x20;

Then we just wait around for the cronjob to run again.

!\[\[37\_Shared\_image0023.png]]

&#x20;

We now have a private rsa key.

!\[\[37\_Shared\_image0024.png]]

&#x20;

!\[\[37\_Shared\_image0025.png]]

&#x20;

PE2:

Now, we can investigate that redis server that is running.

!\[\[37\_Shared\_image0026.png]]

&#x20;

We would need to find some kind of password for the redis-cli, and then we can do stuff like load modules to execute commands as root.

Checking out the /usr/local/bin/redis\_connector\_dev script, we can find this.

!\[\[37\_Shared\_image0027.png]]

&#x20;

This is the password that we need to use here for the Redis instance.

After finding the password, we can just load modules using the redis exploit to gain RCE.

[https://github.com/n0b0dyCN/RedisModules-ExecuteCommand](https://github.com/n0b0dyCN/RedisModules-ExecuteCommand)

Take note that we would have to make the module locally within the machine, as not doing so will result in some kind of error.

!\[\[37\_Shared\_image0028.png]]

&#x20;

!\[\[37\_Shared\_image0029.png]]

&#x20;

Rooted.

&#x20;
