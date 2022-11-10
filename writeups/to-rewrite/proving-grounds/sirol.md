# Sirol

Sirol

Tuesday, 15 March 2022

5:12 pm

Yet another Linux machine.

The target IP is 192.168.73.54

My IP is 192.168.49.73.

&#x20;

!\[\[03\_Sirol\_image001.png]]

Alright, these ports are open, and we can run a more detailed scan through -sV and -O

!\[\[03\_Sirol\_image002.png]]

Take note of port 3306, which has unauthorized or something.

We can't connect to this and there seems to be no other possible enumeration methods for now.

!\[\[03\_Sirol\_image003.png]]

We can't connect to it either.

&#x20;

!\[\[03\_Sirol\_image004.png]]

Let's enumerate the web server.

!\[\[03\_Sirol\_image005.png]]

Simple PHP web calculator.

&#x20;

Looking at the above, I can see there are parameters being attached to the back of the URL when we run any function.

!\[\[03\_Sirol\_image006.png]]

Also, take note of how both of these parameters are num1, meaning that it would just display num1 on the screen.

&#x20;

So this is a basic calculator thing, that gets its results on the server side instead of the client side.

Running a more detailed nmap scan as I may have missed something.

&#x20;

Ran an all port nmap scan.

Forgot to do so earlier, and also that PG is notorious for rabbit holes and stuff.

!\[\[03\_Sirol\_image007.png]]

Oops.

Port 5601 is a new one.

&#x20;

When visited, it seems to be a Kibana website.

!\[\[03\_Sirol\_image008.png]]

Looking at the page source code, we can identify a few things.

!\[\[03\_Sirol\_image009.png]]

This is version 6.5.0.

&#x20;

There is also this port to another IP address here, perhaps pivoting is in need.

!\[\[03\_Sirol\_image0010.png]]

Googling, I found this from Github about [CVE-2019-7609](https://github.com/mpgn/CVE-2019-7609), which is an exploit that affects Kibana verions before 6.6.1.

!\[\[03\_Sirol\_image0011.png]]

Let's try this.

!\[\[03\_Sirol\_image0012.png]]

This should work on a listener port.

&#x20;

After a while, I changed to incognito browsing and also reset the machine in case I was messing up anywhere. Took a while before it starting working as per planned.

&#x20;

Wonder why.

I even found a script to exploit this thing and it was not working.

&#x20;

!\[\[03\_Sirol\_image0013.png]]

&#x20;

This should have worked on this, but instead I get nothing.

I generated a brand new vpn file and tried again.

&#x20;

Right, so I'm just gonna leave this here and the exploit is just not working for me.

&#x20;

!\[\[03\_Sirol\_image0014.png]]

&#x20;

!\[\[03\_Sirol\_image0015.png]]

&#x20;

!\[\[03\_Sirol\_image0016.png]]

&#x20;

Came back to this one, wanted to get it badly.

We can try other ports like port 21.

&#x20;

!\[\[03\_Sirol\_image0017.png]]

Seems like we're in a container.

!\[\[03\_Sirol\_image0018.png]]

&#x20;

&#x20;

Within the root directory, there was this script here.

!\[\[03\_Sirol\_image0019.png]]

&#x20;

From here, we can determine that there is another website connected to us.

!\[\[03\_Sirol\_image0020.png]]

Whatever that URL was, it was our ticket out perhaps.

&#x20;

From here, we can go to the /run directory and find out more about the sockets and stuff.

&#x20;

!\[\[03\_Sirol\_image0021.png]]

Not helpful here.

&#x20;

{width="4.625in" height="1.8125in"}

One of those is able to get us out. And we can even see the mountpoint.

!\[\[03\_Sirol\_image0023.png]]

We were indeed within some disk.

!\[\[03\_Sirol\_image0024.png]]

We can try sda1.

&#x20;

!\[\[03\_Sirol\_image0025.png]]

&#x20;

Once we do this, we would be able to get all of the stuff from the other disk into this directory.

&#x20;

!\[\[03\_Sirol\_image0026.png]]
