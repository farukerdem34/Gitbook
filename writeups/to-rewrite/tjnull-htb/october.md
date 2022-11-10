# October

October

Friday, 11 March 2022

9:45 pm

This is a Linux machine.

The target IP is 10.10.10.16.

My IP is 10.10.16.9.

&#x20;

There's some form of BOF here, which is always good to practice.

Enumeration time.

!\[\[77\_October\_image001.png]]

Alright.

!\[\[77\_October\_image002.png]]

&#x20;

Website:

!\[\[77\_October\_image003.png]]

&#x20;

Looks to be something simple. Let's look around.

!\[\[77\_October\_image004.png]]

Simple little blog.

Ran a quick directory scan and revealed that this is running on PHP.

Did not fare too well with directory enumeration, hence I stopped.

&#x20;

Ran a vuln script and searched for some exploits pertaining to 'Vanilla'. There were plenty!

&#x20;

Created an account first:

!\[\[77\_October\_image005.png]]

&#x20;

When viewing the blog, it sems that I can comment and the view counter is increasing...Does this mean someone is constantly viewing our page?

!\[\[77\_October\_image006.png]]

That link does not work however.

When commenting on anything however, it seems to specify that we can include Markdown syntax.

&#x20;

!\[\[77\_October\_image007.png]]

Googling around, combined with the view counter, it seems that we can execute some form of XSS here.

I ran a nc on port 80 and wanted to experiment by including some form of URL to redirect the user to my server.

&#x20;

!\[\[77\_October\_image008.png]]

&#x20;

This didn't work, but someone was continuously viewing the page, as each time I posted something the view count went by 3. This would mean two other users are on the machine.

&#x20;

However, this does not seem to get me anything else. So I abandoned this idea.

&#x20;

!\[\[77\_October\_image009.png]]

Rainlab.User...

!\[\[77\_October\_image0010.png]]

OctoberCMS...

&#x20;

A quick google search reveals that the server is managed on this /backend directory, and when visited it refers us to a login page.

&#x20;

!\[\[77\_October\_image0011.png]]

I googled for default credentials, which were admin:admin.

&#x20;

!\[\[77\_October\_image0012.png]]

I'm now in the backend portion.

&#x20;

Viewing the media tab, we can upload files here.

!\[\[77\_October\_image0013.png]]

Seems to accept php5 extensions.

&#x20;

I searchsploited this first.

&#x20;

!\[\[77\_October\_image0014.png]]

Saw this little part about extensions:

!\[\[77\_October\_image0015.png]]

Seems to block all except .php5.

&#x20;

Let's try uploading a php5 shell.

{width="5.9375in" height="0.9166666666666666in"}

&#x20;

I could upload this file.

&#x20;

!\[\[77\_October\_image0017.png]]

Now, I knew where to visit it because of the earlier menu link.

!\[\[77\_October\_image0018.png]]

This was where the dr.php5 file was stored. Let's try a simple command.

!\[\[77\_October\_image0019.png]]

Alright, we have RCE.

!\[\[77\_October\_image0020.png]]

&#x20;

Let's try a bash shell.

!\[\[77\_October\_image0021.png]]

Cool!

There was one user called harry, and we are able to get the user flag.

&#x20;

!\[\[77\_October\_image0022.png]]

&#x20;

There was also this tar folder there.

Couldn't do anything with it yet.

I decided to run a quick linpeas to check whether there was any crob job or anything of interest.

&#x20;

There was mysql credentials stored on this interestingly.

!\[\[77\_October\_image0023.png]]

Su does not work. Let's take a look at the database then.

Does not work either.

Moving on!

&#x20;

Looking at SUIDs, I saw this...

!\[\[77\_October\_image0024.png]]

What be this?

&#x20;

This just takes an input and does nothing with it.

Fortunately, gdb is installed on the machine.

&#x20;

Let's gdb this thing.

Looking at the main function, it seems to use strcpy, which indicates an BOF vulnerability. I can already imagine that we need to call the system library location, followed by including shell code in there to gain a root shell.

&#x20;

Right, we need to get this thing onto our machine.

{width="14.354166666666666in" height="8.041666666666666in"}

&#x20;

Ran a checksec on this, and determined that NX was enabled, so this would not be as simple as running some random shellcode. Also

Running a few ldds, we can tell that the memory of the libc changes each time.

&#x20;

!\[\[77\_October\_image0026.png]]

&#x20;

Ran a pattern\_Create with 200 and then ran it with the program.

&#x20;

We can see it crash basically.

&#x20;

!\[\[77\_October\_image0027.png]]

Looking at the EIP, we can determine that we are able to find the offset at 112 bytes.

!\[\[77\_October\_image0028.png]]

&#x20;

Now, from here let's test with a offset of 112 and some random bytes to see if we can change the EIP. I appended 4 Ps to the end and we can see that the EIP is pretty much there.

&#x20;

!\[\[77\_October\_image0029.png]]

&#x20;

From here, it's a simple Return2libc code. Although the memory changes each time, it should eventually hit back the same.

Just need to find the addresses now.

&#x20;

Let's take some random memory locations first.

!\[\[77\_October\_image0030.png]]

This would be libc.

&#x20;

Find the offset for /bin/sh, exit() and then system()

readelf -s /lib/i386-linux-gnu/libc.so.6 | grep -e " system@" -e " exit@"

strings -a -t x /lib/i386-linux-gnu/libc-2.27.so | grep "/bin/sh"

&#x20;

!\[\[77\_October\_image0031.png]]

These would be the exit and system functions:

!\[\[77\_October\_image0032.png]]

&#x20;

Great.

!\[\[77\_October\_image0033.png]]

These would be the shellcodes respectively.

Our payload would be:

Payload = 112 bytes + system() + exit() + /bin/sh

&#x20;

Conveniently, there's python in the machine, so let's just use that.

We know the machine is little-endian, so remember to reverse all of that into the right order.

&#x20;

This would be our command:

!\[\[77\_October\_image0034.png]]

&#x20;

We would just run ovrflw $(python command here) over and over.

I checked my code again, and it turns out there were small errors in it.

&#x20;

After a long time...

!\[\[77\_October\_image0035.png]]

That # means we are root.

&#x20;

&#x20;

&#x20;
