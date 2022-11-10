# Charlotte (walkthrough)

Charlotte (walkthrough)

Wednesday, 30 March 2022

9:37 pm

This is a Linux machine with the following ports open:

!\[\[45\_Charlotte (walkthrough)\_image001.png]]

&#x20;

!\[\[45\_Charlotte (walkthrough)\_image002.png]]

&#x20;

Ports 80 just has a login page that does not seem to go anywhere just yet.

Interestingly, when we browse the page, it shows this interesting header:

!\[\[45\_Charlotte (walkthrough)\_image003.png]]

&#x20;

Seems that the requests are being forwarded to port 7000.

&#x20;

Port 8000 has this authorisation box here.

!\[\[45\_Charlotte (walkthrough)\_image004.png]]

Interesting.

&#x20;

!\[\[45\_Charlotte (walkthrough)\_image005.png]]

There are other things included here as well.

&#x20;

When I see RPC, I tend to want to test what we can mount onto:

!\[\[45\_Charlotte (walkthrough)\_image006.png]]

&#x20;

Turns out we can indeed mount on this, and it has a backups folder here.

!\[\[45\_Charlotte (walkthrough)\_image007.png]]

When mounted we can view these.

Turns out these are the files for the service running on port 8000.

!\[\[45\_Charlotte (walkthrough)\_image008.png]]

&#x20;

We can see that it has a password from the dotenv file, but it seems that we do not have it here.

&#x20;

The JSON files show this:

!\[\[45\_Charlotte (walkthrough)\_image009.png]]

&#x20;

{width="4.864583333333333in" height="1.59375in"}

&#x20;

This is the process of getting the authentication stuff.

So we just need to find out a method of which to bypass this authentication to access the main page.

&#x20;

We can gobuster this to find out more about the website.

!\[\[45\_Charlotte (walkthrough)\_image0011.png]]

&#x20;

There's a README and LICENSE.

!\[\[45\_Charlotte (walkthrough)\_image0012.png]]

&#x20;

So we would need to have our user agent be different.

This uses prerender for bots, which makes it good for us to know.

Afterwards, it talks about basic PHO security, which is weird because PHP Magic hashes could be the thing here.

&#x20;

Anyways, we can look at all of the nginx configurations and be able to work out what's going on.

&#x20;

We can see that basically, we want the condition of prerender to be 1.

!\[\[45\_Charlotte (walkthrough)\_image0013.png]]

&#x20;

!\[\[45\_Charlotte (walkthrough)\_image0014.png]]

&#x20;

And then it would send the request to the respective place.

!\[\[45\_Charlotte (walkthrough)\_image0015.png]]

&#x20;

Interesting.

&#x20;

There was also this admin thing here.

!\[\[45\_Charlotte (walkthrough)\_image0016.png]]

&#x20;

We can quickly bypass this using some SSRF.

&#x20;

We can bypass this via changing the user-agent to googlebot, and the Host to localhost to make it proxy through port 3000.

&#x20;

!\[\[45\_Charlotte (walkthrough)\_image0017.png]]

&#x20;

Then we would get credentials.

Afterwards, we can access the control panel:

!\[\[45\_Charlotte (walkthrough)\_image0018.png]]

&#x20;

From here, we can do some form of prototype pollution that would lead to RCE.

This is because there are two packages being imported within this page.

&#x20;

The first one is merge.js, which is vulnerable to prototype pollution.

!\[\[45\_Charlotte (walkthrough)\_image0019.png]]

&#x20;

The other one is EJS, which is vulnerable to RCE through the prototype pollution of merge.

!\[\[45\_Charlotte (walkthrough)\_image0020.png]]

&#x20;

As we can see, it takes the systems object.

[https://kerupuksambel.medium.com/pwn2win-2021-ctf-writeup-illusion-88f6c7737de3](https://kerupuksambel.medium.com/pwn2win-2021-ctf-writeup-illusion-88f6c7737de3)

&#x20;

This writeup was useful in determining what to do.

&#x20;

Here's an example of the API working properly.

!\[\[45\_Charlotte (walkthrough)\_image0021.png]]

&#x20;

Here's an example of prototype pollution.

!\[\[45\_Charlotte (walkthrough)\_image0022.png]]

&#x20;

Basically, cramming in more objects than intended.

From here, we can get a shell.

We just need to send two requests, the first one above (which basically starts the pollution) and then this one below:

&#x20;

!\[\[45\_Charlotte (walkthrough)\_image0023.png]]

&#x20;

This would give us the shell.

Then reload the page and we should have some form of shell on our listener.

&#x20;

{width="8.229166666666666in" height="2.2291666666666665in"}

&#x20;

Flag:

!\[\[45\_Charlotte (walkthrough)\_image0025.png]]

&#x20;

PE:

We can see that the user sebastian has a cronjob.

!\[\[45\_Charlotte (walkthrough)\_image0026.png]]

&#x20;

This script does some stuff regarding node and imports packages.

!\[\[45\_Charlotte (walkthrough)\_image0027.png]]

&#x20;

We can abuse this by creating some form of package.js shell.

Then this would execute it accordingly.

!\[\[45\_Charlotte (walkthrough)\_image0028.png]]

&#x20;

This would do, then we just wait around.

{width="9.125in" height="2.8125in"}

&#x20;

We are part of the sudo group.

&#x20;

!\[\[45\_Charlotte (walkthrough)\_image0030.png]]

&#x20;

Cool.

&#x20;

Flag:

!\[\[45\_Charlotte (walkthrough)\_image0031.png]]

&#x20;
