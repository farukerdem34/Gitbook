# PlanetExpress

PlanetExpress

Friday, July 1, 2022

1:33 PM

Nmap scan:

!\[\[66\_PlanetExpress\_image001.png]]

&#x20;

!\[\[66\_PlanetExpress\_image002.png]]

&#x20;

This uses Pico CMS, which has some exploits for it.

&#x20;

Port 80:

!\[\[66\_PlanetExpress\_image003.png]]

There isn't much with this website, except for the fact that well, port 9000 seems to be hidden or something.

&#x20;

Gobuster scans:

!\[\[66\_PlanetExpress\_image004.png]]

&#x20;

!\[\[66\_PlanetExpress\_image005.png]]

&#x20;

Twig!

From here, we can sort of find out the version being used.

!\[\[66\_PlanetExpress\_image006.png]]

&#x20;

This is a rather new version, but we know there's twig and there's pico CMS.

&#x20;

We can also find out the different packages that have been installed for this machine.

&#x20;

!\[\[66\_PlanetExpress\_image007.png]]

&#x20;

!\[\[66\_PlanetExpress\_image008.png]]

&#x20;

!\[\[66\_PlanetExpress\_image009.png]]

&#x20;

We don't get much from here.

Let's take a look at the config file.

From github, we can see that /config has a config.yml file and we can try looking at that, of which it exists.

&#x20;

!\[\[66\_PlanetExpress\_image0010.png]]

&#x20;

!\[\[66\_PlanetExpress\_image0011.png]]

&#x20;

We can see that PicoTest is enabled.

!\[\[66\_PlanetExpress\_image0012.png]]

&#x20;

From here, we can get PHPInfo.

!\[\[66\_PlanetExpress\_image0013.png]]

&#x20;

We can see this as well.

From here, we can use hacktricks and think of exploits pertaining to port 9000.

&#x20;

Port 9000:

Firstly, we need to check whether or not there are disabled functions.

!\[\[66\_PlanetExpress\_image0014.png]]

&#x20;

Of which there are a lot. However, passthru is not disabled, and we can use this script to gain RCE.

[https://gist.githubusercontent.com/phith0n/9615e2420f31048f7e30f3937356cf75/raw/ffd7aa5b3a75ea903a0bb9cc106688da738722c5/fpm.py](https://gist.githubusercontent.com/phith0n/9615e2420f31048f7e30f3937356cf75/raw/ffd7aa5b3a75ea903a0bb9cc106688da738722c5/fpm.py)

&#x20;

{width="9.791666666666666in" height="2.7291666666666665in"}

&#x20;

We can confirm this by using id.

{width="9.791666666666666in" height="2.2604166666666665in"}

&#x20;

We now have RCE, now we just need to gain a shell.

{width="9.84375in" height="1.0in"}

&#x20;

{width="7.302083333333333in" height="1.9583333333333333in"}

&#x20;

Flag:

!\[\[66\_PlanetExpress\_image0019.png]]

&#x20;

We have a user called astro here, and the next step should be going becoming that user.

&#x20;

PE:

I ran LinPEAS.

!\[\[66\_PlanetExpress\_image0020.png]]

there's this weird binary.

&#x20;

{width="9.854166666666666in" height="2.4166666666666665in"}

&#x20;

I don't know what it really does...

&#x20;

{width="7.895833333333333in" height="4.833333333333333in"}

I think this can be used to read files.

&#x20;

{width="7.84375in" height="1.84375in"}

It seems to make files readable.

&#x20;

From here, we can get the hashes of the astro and root user.

Cracking:

{width="9.760416666666666in" height="3.9791666666666665in"}

&#x20;

!\[\[66\_PlanetExpress\_image0025.png]]

&#x20;

Su:

!\[\[66\_PlanetExpress\_image0026.png]]

&#x20;

Flag:

!\[\[66\_PlanetExpress\_image0027.png]]

&#x20;

&#x20;

&#x20;
