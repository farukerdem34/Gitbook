# Doctor

Doctor

Wednesday, 16 February 2022

5:24 pm

This is a Linux machine.

My IP is 10.10.16.9.

The target IP is 10.10.10.209.

&#x20;

Let's do a quick enumeration of this. Take note of port 8089.

!\[\[42\_Doctor\_image001.png]]

Cool little website.

!\[\[42\_Doctor\_image002.png]]

&#x20;

I ran a directory enumeration while looking through the website for anything interesting.

!\[\[42\_Doctor\_image003.png]]

&#x20;

This revealed nothing of interest. However, there is a little hint here.

&#x20;

!\[\[42\_Doctor\_image004.png]]

I added doctors.htb into my /etc/hosts folder, and managed to access this page.

!\[\[42\_Doctor\_image005.png]]

So doctors.htb is another website that we should check out. The website presented me with a login page and an option to create an account. I logged in afterwards.

&#x20;

!\[\[42\_Doctor\_image006.png]]

&#x20;

Once I was in, I noticed that there was this Message board ( you can see my posts), and I tried all forms of payloads.

I created a bunch of posts and none of my SSTI payloads were working.

&#x20;

I inspected the page source and came across this little gem.

!\[\[42\_Doctor\_image007.png]]

&#x20;

I loaded in the browser but it did not work. Hence I tried loading it in Burpsuite and it returned some interesting information.

&#x20;

!\[\[42\_Doctor\_image008.png]]

The 49 there means that SSTI indeed does exist on the page! This is a proof of concept. The next step would be to try and get an error to see any other information regarding the web engine.

&#x20;

!\[\[42\_Doctor\_image009.png]]

This payload did not work, so I tried with \{{7\*7\}}.

&#x20;

This is the payload that generates an SSTI.

&#x20;

Anyways I tried \{{7/0\}} and this resulted in a bunch of errors, and I had to change the account.

Anyways, there has to be a way to determine what kind of version I'm working with without crashing the /archive.

Going on hacktricks, this was a useful diagram to have.

!\[\[42\_Doctor\_image0010.png]]

&#x20;

I know that \{{7\*7\}} was the payload that worked, hence I tried the next thing \{{7\*'7'\}}. This produced results that point me to some form of engine.

&#x20;

!\[\[42\_Doctor\_image0011.png]]

Seems that it's either Jinja2 or Twig based on what I sent.

&#x20;

Anyways, it is possible to get me a reverse shell from this.

There's a payload on PayloadsAllTheThings which can help me out.

!\[\[42\_Doctor\_image0012.png]]

This should work.

Take note that in the original payload, the code that they execute is to cat flag.txt. Change that to /bin/bash and append the -I flag to get a shell instead.

Slap that payload into the title, and get the /archive.

!\[\[42\_Doctor\_image0013.png]]

And we're in.

Seems that we're the user web, and we have no access to user shaun.

!\[\[42\_Doctor\_image0014.png]]

There's this blog.sh script within our directory, and this is the output.

&#x20;

!\[\[42\_Doctor\_image0015.png]]

&#x20;

I didn't know what to do, hence I downloaded some LinPeas to get me some working stuff.

&#x20;

LinPeas is always so cute to me.

!\[\[42\_Doctor\_image0016.png]]

&#x20;

Anyways, I ran it and looked through the files and binary and all that junk. At the same time, I knew this was an apache2 server, hence I wanted to take a look around inside the thing if I could find anything that was of interest.

!\[\[42\_Doctor\_image0017.png]]

This one line stuck out, and Guitar123 does not look like an email.

So turns out, that's the user password.

&#x20;

!\[\[42\_Doctor\_image0018.png]]

Cool.

&#x20;

Anyways Shaun may not run any form of sudo on this machine. Sad.

From here, I don't think there's anything else of interest within the machine. Nothing stands out to me.

&#x20;

I was at a wall, and then I remembered that the Splunk server still exists there.

For some reason, the splunk server kept resetting my connection. Id don't know what's going on there.

&#x20;

Anyways I looked around the machine and found the splunk version, which was 8.0.5.

!\[\[42\_Doctor\_image0019.png]]

Cool. So turns out there are exploits for this when I googled online.

This would require a script called PySplunkWhisperer.

Using this, we need a username and password. I suppose our friend shaun would just reuse the same old one.

&#x20;

Using this script and a quick reverse shell, this got me another reverse shell.

&#x20;

!\[\[42\_Doctor\_image0020.png]]

I did not expect it to be the root shell LOL.

!\[\[42\_Doctor\_image0021.png]]

Well, that was pwned.

&#x20;

I looked up a guide as to why my initial exploit was not working, and why we need to append the bash -c.

So as it turns out, the -c flag means that commands are read from the first non-option argument command\_string.

&#x20;

So this means that the bash -c before the actual reverse shell would actually just cause the shell to spawn first, then start reading the command.

&#x20;

I assume this is the case because of the fact that in most RCEs, there isn't a shell just waiting for us right there.

&#x20;

Take note that we have to append this flag next time we encounter our bash script not working as planned.
