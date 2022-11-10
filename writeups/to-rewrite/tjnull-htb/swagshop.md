# SwagShop

SwagShop

Sunday, 6 February 2022

4:11 pm

This is a Linux machine.

The target IP is 10.10.10.140.

My IP is 10.10.16.5.

&#x20;

Early scan reveals there is SSH and HTTP present on the machine.

More detailed scan reveals this.

&#x20;

!\[\[19\_SwagShop\_image001.png]]

&#x20;

Cool! When trying to enter the website, be sure to add to /etc/hosts.

&#x20;

This shows us there is some kind of shop present.

!\[\[19\_SwagShop\_image002.png]]

&#x20;

Gobuster reaveals some interesting directories.

&#x20;

!\[\[19\_SwagShop\_image003.png]]

&#x20;

!\[\[19\_SwagShop\_image004.png]]

&#x20;

I noticed that there was a 2014 version of Magento being used, and I'm sure that current versions are not that anymore.

Searched around searchsploit for magento related vulnerabilities and found these.

&#x20;

!\[\[19\_SwagShop\_image005.png]]

&#x20;

Seems that the RCE should be severe enough, let's try it. Didn't work :(

&#x20;

So far I only know that the Magento exploits should work, because this version is way old.

&#x20;

Found a few exploits regarding this, and tried a lot of them. [This](https://github.com/joren485/Magento-Shoplift-SQLI/blob/master/poc.py) particular one worked out well.

&#x20;

!\[\[19\_SwagShop\_image006.png]]

&#x20;

Cool, now I have admin panel access. Looking at it does not give me much though.

&#x20;

But I realise that this gives me immediate ability to break into the website through the use of Authenticated RCE.

&#x20;

!\[\[19\_SwagShop\_image007.png]]

&#x20;

This should work, but we need some stuff before we can use it.

&#x20;

!\[\[19\_SwagShop\_image008.png]]

&#x20;

We have a username and password from the other exploit, which we can use on /index.php/admin.

!\[\[19\_SwagShop\_image009.png]]

&#x20;

Now we just need to find some kind of install date. The directory given by the exploit works.

!\[\[19\_SwagShop\_image0010.png]]

&#x20;

&#x20;
