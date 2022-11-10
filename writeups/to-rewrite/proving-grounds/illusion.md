# Illusion

Illusion

Sunday, July 3, 2022

12:09 PM

Nmap scan:

!\[\[68\_Illusion \_image001.png]]

&#x20;

Port 80:

Just a standard company web page.

!\[\[68\_Illusion \_image002.png]]

&#x20;

There are some usernames here.

Nothing much though.

&#x20;

There is also a login and dashboard.php.

!\[\[68\_Illusion \_image003.png]]

&#x20;

Simple credentials don't work here.

&#x20;

Let's think of it as a PHP file that we need to somehow access.

&#x20;

Because we need to compare hashes, we cannot have it as a string. Let's remove the quotes and send the hash as it is.

!\[\[68\_Illusion \_image004.png]]

&#x20;

Loading this request in the browser would show us the dashboard.php.

!\[\[68\_Illusion \_image005.png]]

&#x20;

And there is this query thingy.

&#x20;

!\[\[68\_Illusion \_image006.png]]

&#x20;

We can attempt some SSTI, because we know that this is a PHP app (perhaps running twig)

&#x20;

!\[\[68\_Illusion \_image007.png]]

&#x20;

!\[\[68\_Illusion \_image008.png]]

&#x20;

This confirms there is indeed SSTI, and we can use this to gain a shell.

&#x20;

RCE:

!\[\[68\_Illusion \_image009.png]]

&#x20;

!\[\[68\_Illusion \_image0010.png]]

&#x20;

We can do stuff like read the /etc/passwd file.

&#x20;

!\[\[68\_Illusion \_image0011.png]]

&#x20;

And we know the user is called james.

&#x20;

We can just input a shell to become www-data.

!\[\[68\_Illusion \_image0012.png]]

&#x20;

{width="8.53125in" height="2.25in"}

&#x20;

Upgrade shell:

{width="8.354166666666666in" height="2.1979166666666665in"}

&#x20;

From here, we can focus on PE.

&#x20;

Flag:

!\[\[68\_Illusion \_image0015.png]]

&#x20;

PE:

Potential password:

!\[\[68\_Illusion \_image0016.png]]

&#x20;

Within the user james's home directory, there is this weird redis-openssl-gen-pass.txt.

!\[\[68\_Illusion \_image0017.png]]

&#x20;

And we can find a redis instance listening within the machine itself.

!\[\[68\_Illusion \_image0018.png]]

&#x20;

We can forward that out and then connect to it, although I'm not sure what I'm supposed to do with this openssl-gen-pass.

&#x20;

Port Fowarding with Chisel.

!\[\[68\_Illusion \_image0019.png]]

&#x20;

However, trying to connect to this instance requires a password of some sorts.

!\[\[68\_Illusion \_image0020.png]]

&#x20;

Furthermore, this is not a vulnerable version of redis.

&#x20;

The redis conf file is not accessible by us, and all we have is that genpass file.

!\[\[68\_Illusion \_image0021.png]]

&#x20;

Perhaps we are supposed to use OpenSSL to generate a password.

This was a key of some sorts, and we have to use this somehow.

&#x20;

Before proceeding with this, I ran linPEAS on the machine to get a better picture of what else we can do.

&#x20;

LinPEAS:

We can see that root is the one running this server, meaning once we find this password, we can load modules to do RCE as root.

!\[\[68\_Illusion \_image0022.png]]

&#x20;

I had a suspicion that we have to use this thing somehow to generate a form of password or something.

&#x20;

The simplest explanation tends to work hwoever.

!\[\[68\_Illusion \_image0023.png]]

&#x20;

We can now enumerate this thing a little bit more.

What we can do now is load modules into this.

&#x20;

[https://github.com/n0b0dyCN/RedisModules-ExecuteCommand](https://github.com/n0b0dyCN/RedisModules-ExecuteCommand)

&#x20;

{width="9.822916666666666in" height="2.15625in"}

&#x20;

We now have RCE as root.

!\[\[68\_Illusion \_image0025.png]]

&#x20;

{width="7.239583333333333in" height="1.4583333333333333in"}

&#x20;

!\[\[68\_Illusion \_image0027.png]]

&#x20;

Cool.

&#x20;

Flag:

!\[\[68\_Illusion \_image0028.png]]

&#x20;

&#x20;
