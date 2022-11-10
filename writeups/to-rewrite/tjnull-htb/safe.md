# Safe

Safe

Wednesday, 9 March 2022

1:58 pm

This is a Linux machine.

The target IP is 10.10.10.147.

My IP is 10.10.16.9.

&#x20;

Enumeration time.

!\[\[72\_Safe\_image001.png]]

While waiting for the detailed scan, went to port 80 and port 1337 knowing that they are likely some kind of website.

&#x20;

!\[\[72\_Safe\_image002.png]]

&#x20;

Port80:

!\[\[72\_Safe\_image003.png]]

&#x20;

Port 1337:

!\[\[72\_Safe\_image004.png]]

Port 1337 is clearly more interesting, wonder what we can do on this.

&#x20;

Let's try a netcat to connect to this.

!\[\[72\_Safe\_image005.png]]

Hmm. This is some kind of echoer.

&#x20;

Tried this string:

!\[\[72\_Safe\_image006.png]]

Sending this would crash the server. This means that there is no user input validation on the content that we fit into the website.

&#x20;

Inspecting the web page on port 80, we can see this:

&#x20;

!\[\[72\_Safe\_image007.png]]

Myapp...

&#x20;

Visiting /myapp would make us download an application.

This is clearly a RE challenge for us and we need to do some ROP, hopefully not ROP gadgets.

&#x20;

For now, let's IDA this thing.

!\[\[72\_Safe\_image008.png]]

Okay, so this seems to call system, and there seems to be a length of a string here.

Based on my understanding, this uses the gets function, which is notorious for not validating the length of user input. This would result in there being the buffer overflow that we need to execute.

&#x20;

First of all, let's determine what is the length of the offset so we can find the instruction pointer to change that.

&#x20;

From here, I determined that we should run gdb-peda which is better for ROP related exploits.

Let's first run a checksec to see what are the things.

!\[\[72\_Safe\_image009.png]]

NX is enabled here, hence the stack might not be executable when on the webapp.

&#x20;

For some reason, the application is not letting me enter my stuff, hence we need to enter this command, which would make gdb follow the parent:

**set follow-fork-mode parent.**

&#x20;

From here, we should be able to load in whatever we want.

Let's create a pattern of 200 and then run it with that.

{width="11.1875in" height="9.5in"}

This would cause the stack to overflow with characters.

Now, we need to take note of the first 4 bytes of data that it overflows. This one in particular:

&#x20;

!\[\[72\_Safe\_image0011.png]]

This is the string that overwrites the instruction pointer and makes everything go haywire as it cannot return properly.

&#x20;

Now let's investigate the offset of 120 and add 8 more characters after to see if I can change the pointers.

!\[\[72\_Safe\_image0012.png]]

As we can see, I am able to create an offset of 120 junk characters and then use these 8 to overflow the rest of the program.

&#x20;

So now, we know that there is the offset of 120. Let's go see what functions exist within the code.

When looking at IDA, interestingly, there is the system present within the code. Then, we also can see another function called test.

!\[\[72\_Safe\_image0013.png]]

The system function would make it such that we do not need to know where the system function is stored within the libc libraries, which makes it easier to understand.

&#x20;

!\[\[72\_Safe\_image0014.png]]

Right, that highlighted memory address is basically never called, and it looks like a good thing to use for our ROP.

&#x20;

So right now let's take a look at our possible payload.

Payload = junk (120 bytes) + 4 bytes of new return pointer(where system is located within the code) + 4 bytes of calling /bin/sh + rest of my shell

&#x20;

Within the main code, it seems to call the system function.

!\[\[72\_Safe\_image0015.png]]

When disassembled in gdb:

{width="7.75in" height="4.239583333333333in"}

The system function is located at 0x401040.

&#x20;

Right, now for ROP we need something called a gadget. Before that, we need to somehow jump there.

Let's take a look at the test function.

!\[\[72\_Safe\_image0017.png]]

This moves whatever is in rsp to to rbp and rdi, and we can only benefit from this if we are able to control rbp and rdi.

!\[\[72\_Safe\_image0018.png]]

This would be the gadget we are using. This controls r13 and r14 and r15, which is something we have to put up with. If there was another gadget that we can just use to control r13, we would use that for sure.

&#x20;

Running the tool again:

!\[\[72\_Safe\_image0019.png]]

&#x20;

We can see that the RBP is the 8 bytes of 1, and this comes after junk of 112 bytes.

The RIP is not touched, hence anything else after this would change the RIP. We don't want that yet.

So our payload is looking like this:

&#x20;

Payload = junk (112 bytes) + (/bin/sh) + (8 bytes for gadget) + (pop r13 with system address) + (16 bytes to fill in r14 and r15) + (address of test(), which would cause the jump to r13 with the system address)

&#x20;

Let's use pwntools to create a quick script which would trigger the /bin/sh to happen.

Since this is using more general purpose registers of r13, we know that this is a 64-bit machine.

This is what our payload would look like:

!\[\[72\_Safe\_image0020.png]]

For now, this should work.

&#x20;

Modified my script a bit, and this is the end result. I took note that we could directly call stuff using the ELF and ROP of the file.

{width="10.25in" height="5.59375in"}

&#x20;

Ran it and we got a shell.

!\[\[72\_Safe\_image0022.png]]

&#x20;

Grab the user flag, and also echo in our public key into authorized\_keys so we can SSH in.

!\[\[72\_Safe\_image0023.png]]

&#x20;

{width="8.447916666666666in" height="3.9479166666666665in"}

&#x20;

When we are in, we can see that there a lots of images and other things like MyPasswords files:

!\[\[72\_Safe\_image0025.png]]

&#x20;

MyPasswords.kdbx is something that is the most interesting.

&#x20;

Transferred all the files over.

!\[\[72\_Safe\_image0026.png]]

&#x20;

There is a keeppass2john.py that we can use to decrypt that file.

!\[\[72\_Safe\_image0027.png]]

Ran a john on this, and it seems that I am unable to crack it...

&#x20;

Hmm. There has to be something to do with those other JPGs.

Bit of googling reveals that we are able to use these images as keyfiles for our hashing.

Took a hint from a walkthrough and it was revealed that I needed to use more than just rockyou, as we needed rockyou-70.txt.

We are to try all images and eventually crack the password:

{width="9.510416666666666in" height="2.25in"}

Once we have this password, we can just use the image as the key and the master password.

&#x20;

!\[\[72\_Safe\_image0029.png]]

Within the entries, we can see Root password.

!\[\[72\_Safe\_image0030.png]]

&#x20;

!\[\[72\_Safe\_image0031.png]]

Darn.

&#x20;

We need to find a way to view this thing.

We can append the -f flag to view it.

!\[\[72\_Safe\_image0032.png]]

Using this password, we can just su to root.

&#x20;

!\[\[72\_Safe\_image0033.png]]

&#x20;

That ROP was challenging.

&#x20;
