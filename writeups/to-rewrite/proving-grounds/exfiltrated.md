# Exfiltrated

Exfiltrated

Wednesday, 23 March 2022

8:47 pm

A linux machine with these ports.

&#x20;

!\[\[38\_Exfiltrated\_image001.png]]

&#x20;

The web page has a log in page, and the usage of admin:admin is enough.

!\[\[38\_Exfiltrated\_image002.png]]

&#x20;

There's a /panel directory

!\[\[38\_Exfiltrated\_image003.png]]

&#x20;

!\[\[38\_Exfiltrated\_image004.png]]

There are a couple of exploits related to this.

We would be using the Arbitrary File Upload thing, and from there, we can gain a shell.

!\[\[38\_Exfiltrated\_image005.png]]

&#x20;

We can get a python3 shell from here.

{width="11.270833333333334in" height="0.8333333333333334in"}

&#x20;

There's one user called coaran here.

Ran a linpeas to enumerate for me.

!\[\[38\_Exfiltrated\_image007.png]]

It seems that the root user is running one cron job here.

&#x20;

!\[\[38\_Exfiltrated\_image008.png]]

This seems to use exiftool, and I wonder if there are ways to inject codes into the images that we put there.

&#x20;

[This](https://github.com/convisolabs/CVE-2021-22204-exiftool) explains how it works. Basically, we are able to embed stuff within our images to execute commands. Since it's a cronjob by root, this is effectively a good way to gain a reverse shell back to us.

It even comes with instructions.

&#x20;

Just get any image file, get their configfile and run the exploit script given. This would embed a base64 code within our image to be executed later by exiftool.

&#x20;

All we need to do is just get the file, and put it within the folder specified in subrion.

!\[\[38\_Exfiltrated\_image009.png]]

&#x20;

Now, we just have a listener shell and wait. Once the cronjob runs, we would have a root shell.

&#x20;

!\[\[38\_Exfiltrated\_image0010.png]]

&#x20;

!\[\[38\_Exfiltrated\_image0011.png]]

&#x20;

!\[\[38\_Exfiltrated\_image0012.png]]

&#x20;
