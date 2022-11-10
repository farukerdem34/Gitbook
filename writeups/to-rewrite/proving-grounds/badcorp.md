# BadCorp

BadCorp

Sunday, June 19, 2022

5:28 PM

A very hard Linux machine that has these services running:

&#x20;

!\[\[48\_BadCorp\_image001.png]]

&#x20;

Interestingly, the FTP server specifically states that this is a private system without any anonymous logins allowed.

&#x20;

{width="6.6875in" height="1.8645833333333333in"}

&#x20;

Going onto port 80, we see another basic Corporate portal:

!\[\[48\_BadCorp\_image003.png]]

&#x20;

These are the team members that we are concerned with.

The webpage seems to be running on HTML and not PHP or something.

&#x20;

That's about all we have. For now, we can try brute forcing that log in with some default credentials that normally work. It seems that privsep TLS.

We should try to brute force this FTP, and for wordlists, since this says it is a protected server, we can Cewl the website here.

&#x20;

We can try making usernames with the names we have there using john. Afterwards, we can either use the default FTP Creds Cewl Wordlists. I tried both to be safe.

&#x20;

Eventually, I determined that it was quite pointless to keep brute forcing and waiting like this. I took the walkthrough to determine the first portion only as this was something that wasn't exactly stimulating.

hoswald:34566550

&#x20;

This was the password and the FTP credentials we needed.

Once we are in, we have an encrypted RSA private key. This can be decrypted easily using John.

!\[\[48\_BadCorp\_image004.png]]

&#x20;

Once we have that, we can grab the user flag.

This only shows us that every little bit of information is useful when it comes to pentesting accounts and creating our own wordlists and stuff like that.

!\[\[48\_BadCorp\_image005.png]]

&#x20;

PE:

Within the machine, there is one weird little binary here that seems to backup code.

LinPeas output:

!\[\[48\_BadCorp\_image006.png]]

&#x20;

There is this usr/local/bin/backup file with the SUID binary set.

When running it, it requires a password.

!\[\[48\_BadCorp\_image007.png]]

&#x20;

We can transfer it back to my machine using some base64.

!\[\[48\_BadCorp\_image008.png]]

&#x20;

{width="4.8125in" height="2.8854166666666665in"}

&#x20;

File analysis:

{width="9.541666666666666in" height="1.3020833333333333in"}

&#x20;

We would need to analyse this file and break down the code in order to understand what it does and what this password is.

Ghidra is the tool here.

!\[\[48\_BadCorp\_image0011.png]]

&#x20;

When decompiled, we can see the main function.

!\[\[48\_BadCorp\_image0012.png]]

&#x20;

There are 3 functions being called, **namely the CHECK,SECURITY and COPY functions.**

From security and check come two variables, and should there be no errors, the binary can execute.

&#x20;

Copy Function:

{width="4.177083333333333in" height="2.9375in"}

&#x20;

!\[\[48\_BadCorp\_image0014.png]]

Within the copy function, we can see that we are able to write to two folders here. Interesting again. We will keep this in mind. It seems to copy everything with the -rf flag and then proceed to do something else.

&#x20;

Security Function:

{width="4.0in" height="6.125in"}

There seem to be some characters which cannot be used no matter what.

We can keep a note of these characters that aren't allowed.

Bad Characters:

..

/

|

<

\>

&

\<space>

'

\-

%

&#x20;

They failed to ban $() though! So there might be some code execution after all!

We can test this:

{width="3.9479166666666665in" height="0.84375in"}

&#x20;

{width="3.96875in" height="1.0208333333333333in"}

&#x20;

So there is no protection over $(), hence allowing bash expressions to be executed.

&#x20;

Check Function:

{width="4.5in" height="4.1875in"}

&#x20;

We can see that there is some variables being defined, and then some math occurs with the XOR-ing of the local\_c variable with 0xc.

&#x20;

Finding the Password:

The first step would be to find the password of which this binary uses.

So we can see that at the end, it would take check whether the local\_la variable matches the pw variable basically.

!\[\[48\_BadCorp\_image0019.png]]

&#x20;

We can find this easily.

Then, we need to see how the local\_la variable is being manipulated.

In short, it seems that this is the end goal here, and that every single character of the local\_la variable is being XOR'd with 0xc.

&#x20;

We can reverse this process by trying to figure out what is local\_la at that moment and see how it relates to the password.

{width="4.125in" height="0.7604166666666666in"}

&#x20;

It seems that local\_la is first equated to param\_1 and has a size of 9 bytes. The function takes can take in one argument from main() but there are none passed in.

!\[\[48\_BadCorp\_image0021.png]]

&#x20;

Anyways, it's likely that the original password is passed through this function and then checked with the final value of pw.

We can reverse this portion.

&#x20;

RE:

We can first create a python script to XOR stuff together.

!\[\[48\_BadCorp\_image0022.png]]

This would XOR the stuff that we want together, now we would need to create a for loop to XOR each character with 0xc. This is my finished code, and the password I got:

!\[\[48\_BadCorp\_image0023.png]]

&#x20;

{width="4.0625in" height="0.7916666666666666in"}

This works! Meaning we have found the right password. Cool.

&#x20;

Back to machine:

!\[\[48\_BadCorp\_image0025.png]]

&#x20;

We can now execute the code, and we can see that it takes one file and does something else.

Now, we can begin to think about how to inject some code into this.

We can write to the id\_rsa file.

!\[\[48\_BadCorp\_image0026.png]]

&#x20;

We have FTP access to this machine, and we can write files using that instead.

!\[\[48\_BadCorp\_image0027.png]]

&#x20;

!\[\[48\_BadCorp\_image0028.png]]

&#x20;

Through experimentation with my own shell, I found that using this method works in executing other commands. This is because perhaps there are no regulations over the expression characters we identified earlier(I think?)

{width="3.9479166666666665in" height="0.6041666666666666in"}

&#x20;

!\[\[48\_BadCorp\_image0030.png]]

&#x20;

Cool, so we have found a way to execute commands as root.

&#x20;

{width="3.9791666666666665in" height="0.5625in"}

&#x20;

!\[\[48\_BadCorp\_image0032.png]]

&#x20;

Cool, we are now root, but we have little functions in this shell.

We can escape with nc.

!\[\[48\_BadCorp\_image0033.png]]

&#x20;

{width="6.833333333333333in" height="1.5625in"}

&#x20;

Flag:

!\[\[48\_BadCorp\_image0035.png]]

af4184985c3b7e42774f7daac9dfa292

&#x20;
