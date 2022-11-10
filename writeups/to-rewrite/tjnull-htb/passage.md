# Passage

Passage

Sunday, 20 February 2022

2:10 pm

This is a Linux box.

The Target IP is 10.10.10.206.

My IP is 10.10.16.9.

&#x20;

Enumeration up.

!\[\[44\_Passage\_image001.png]]

!\[\[44\_Passage\_image002.png]]

&#x20;

!\[\[44\_Passage\_image003.png]]

Okay, this we page has different kinds of news or whatever. What's interesting is that the admin has inputted some form of posts. There's this email link present on the page. Upon viewing the page source, this is what we see.

&#x20;

!\[\[44\_Passage\_image004.png]]

&#x20;

Seems that nadiv is the admin or something. Good to take note of.

!\[\[44\_Passage\_image005.png]]

There's also a paul user that is present.

&#x20;

This rest of the websites are to generic names like something@example.com, only these two have special emails. Good to take note of. Additionally, there is a web engine at the bottom.

&#x20;

!\[\[44\_Passage\_image006.png]]

&#x20;

This is powered by cutephp, which has a few exploits that are possible.

&#x20;

We just need to find the right version that is running on this.

Looking at the first post, which is the only post that is actual English, it seems that the web server blocks excessive traffic. This explains why my directory exploits did not work well.

&#x20;

!\[\[44\_Passage\_image007.png]]

Based on the date that this website came out, which is 2020 as stated at the bottom of the page and the cutenews being present, I tried a few exploits.

&#x20;

This exploit worked out well for me.

!\[\[44\_Passage\_image008.png]]

&#x20;

!\[\[44\_Passage\_image009.png]]

&#x20;

From here, I created a quick reverse shell and connected back to my machine.

&#x20;

!\[\[44\_Passage\_image0010.png]]

&#x20;

!\[\[44\_Passage\_image0011.png]]

Ez. As expected, there are two users on this machine.

&#x20;

!\[\[44\_Passage\_image0012.png]]

I don't have access to either one of them, so let's take a look within the www folder to see if there's any form of password or database files.

&#x20;

Taking a look at this file here, I found this load of stuff, which I'm not sure what it is about.

{width="9.729166666666666in" height="3.5104166666666665in"}

&#x20;

This is clearly base64, and decoding it gives me this.

!\[\[44\_Passage\_image0014.png]]

&#x20;

There's some form of deserialization that could occur here, so I took a look at more of those files.

&#x20;

There's the longest one, which was the most suspicious to me.

&#x20;

!\[\[44\_Passage\_image0015.png]]

Decoding it reveals this.

!\[\[44\_Passage\_image0016.png]]

&#x20;

Seems that this is a 64-bit thing, no idea what is it. It's 64-bytes long, hence I suspected that this could potentially be something like SHA-512.

&#x20;

I took a while to collate the hashes that were relevant to me, which were nadev and paul's.

I took paul's and headed to crackstation. This was the result.

!\[\[44\_Passage\_image0017.png]]

&#x20;

!\[\[44\_Passage\_image0018.png]]

Seems that the password is atlanta1. The hash for nadav did not work.

This did the trick.

!\[\[44\_Passage\_image0019.png]]

&#x20;

Grab the user flag while we're at it.

&#x20;

Now, we need to somehow get into other users.

I ran linpeas the machine, and it revealed some interesting information.

&#x20;

!\[\[44\_Passage\_image0020.png]]

Why is there a key for nadav located within paul's keys?

&#x20;

So this would mean that nadav's key is located in paul's directory. I just SSHed from paul straight into nadav and this functioned well.

!\[\[44\_Passage\_image0021.png]]

&#x20;

Ran linpeas again as nadav.

&#x20;

This time, it returned this.

!\[\[44\_Passage\_image0022.png]]

Now things are interesting.

!\[\[44\_Passage\_image0023.png]]

Taking a closer look at CVE-2021-4034, it seems to be a vulnerability found polkit's pkexec utility.

This did not seem to work at all, so back to the drawing board.

&#x20;

There's other files that were flagged out.

!\[\[44\_Passage\_image0024.png]]

There's this USBCreator thingy.

&#x20;

Googling around led me to this [page](https://rioasmara.com/2021/07/16/usbcreator-d-bus-privilege-escalation/).

This would basically extract the private key of the root account and allow us to ssh in.

!\[\[44\_Passage\_image0025.png]]

&#x20;

!\[\[44\_Passage\_image0026.png]]

&#x20;
