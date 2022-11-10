# Love

Love

Sunday, 20 February 2022

7:52 pm

This is a Windows machine.

The target IP is 10.10.10.239.

My IP is 10.10.16.9.

&#x20;

Enumeration time.

Early and fast scan reveals http, msrpc, netbios, https, mysql and upnp, wsman, and pando-pub.

&#x20;

!\[\[46\_Love\_image001.png]]

&#x20;

This is the website. In port 80. Take note of the other HTTP ports that are open, which are likely used for some form of transport layer.

!\[\[46\_Love\_image002.png]]

&#x20;

It would seem that I have no access to the https website, says forbidden.

!\[\[46\_Love\_image003.png]]

&#x20;

At least we can take note of the apache server, and that this is a 64-bit machine.

The SMB ports yield no interest by not allowing any form of null session or enumeration.

!\[\[46\_Love\_image004.png]]

&#x20;

I did a directory scan on the websites I have found first.

We could be looking for some form of mysql credentials later on.

!\[\[46\_Love\_image005.png]]

There seem to be a lot of directories...

&#x20;

All of them redirect us back to the main browser, or they are just not available. This led me to want to try some kind of brute force attempt.

&#x20;

Anyways I took a look at the certificate of the HTTPS website, and found another possible host.

&#x20;

!\[\[46\_Love\_image006.png]]

&#x20;

Time to add this to /etc/hosts and begin to enumerate it.

&#x20;

!\[\[46\_Love\_image007.png]]

&#x20;

This screams RCE. But it is not. I tried it with my own python server, and it returned this.

!\[\[46\_Love\_image008.png]]

&#x20;

!\[\[46\_Love\_image009.png]]

So this works, let's try [http://localhost](http://localhost).

Returned nothing of interest, so I began testing out ports until port 5000 revealed something very cool.

!\[\[46\_Love\_image0010.png]]

&#x20;

Alright, now we have the Voting system Admin password and stuff.

This did not work however, so I tried other methods.

&#x20;

!\[\[46\_Love\_image0011.png]]

There's an unauthenticated RCE, so let's try using that first.

&#x20;

Plop in our details as shown:

!\[\[46\_Love\_image0012.png]]

Based on my directory enumeration, also remove all the votesystem from the URLs, they do not exist in this website I think.

&#x20;

From there, we can run it and this is the outcome.

!\[\[46\_Love\_image0013.png]]

&#x20;

Cool. Grab the user flag.

Check out the systeminfo, seems to be Windows 10 Pro and a 64-bit computer.

Let's transfer over winPEAS and stuff to enumerate for us.

Use the certutil method to bring it over.

&#x20;

Anyways, here are the interesting things that I did research on.

{width="11.697916666666666in" height="4.90625in"}

&#x20;

!\[\[46\_Love\_image0015.png]]

&#x20;

!\[\[46\_Love\_image0016.png]]

The last picture is the most significant, as it allows us to hijack some DLL libraries that can be used to install and execute code as the administrator. In theory, we should be able to spawn a reverse shell. In this case, we can use MSFvenom to create a quick payload.

&#x20;

So from here, we just do the following.

{width="11.541666666666666in" height="1.6770833333333333in"}

&#x20;

Set up a listener port and a HTTP server.

!\[\[46\_Love\_image0018.png]]

Run that msi thing, and on the listener port we should get this.

&#x20;

!\[\[46\_Love\_image0019.png]]

&#x20;

On the listener port, we are now the administrator user and can get into other machines that are connected to this one via wifi or something.

&#x20;
