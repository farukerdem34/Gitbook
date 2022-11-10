# BountyHunter

BountyHunter

Monday, 4 April 2022

5:47 pm

This is a Linux machine with the following services open:

!\[\[15\_BountyHunter\_image001.png]]

&#x20;

It seems that this is some sort of page of which we can submit stuff to.

!\[\[15\_BountyHunter\_image002.png]]

&#x20;

!\[\[15\_BountyHunter\_image003.png]]

The portal button in the top right indicates this.

&#x20;

And going to that page indicated brings us to this little page.

&#x20;

!\[\[15\_BountyHunter\_image004.png]]

&#x20;

When we key in stuff, it seems to print it back out.

&#x20;

!\[\[15\_BountyHunter\_image005.png]]

&#x20;

!\[\[15\_BountyHunter\_image006.png]]

The data here is the post request sent, and it seems to output it in base64 encoded things.

!\[\[15\_BountyHunter\_image007.png]]

&#x20;

Seems this uses XML, which gives me a hint that this might use XXE injections.

Perhaps we could inject some malicious payloads into this through base64.

&#x20;

I was unable to replicate anything of use using this however.

&#x20;

Seeing that there was stuff on the website that was in php, I ran a gobuster with the PHP extensions and found some more interesting things.

!\[\[15\_BountyHunter\_image008.png]]

&#x20;

Within the resources directory, there was a README.txt.

!\[\[15\_BountyHunter\_image009.png]]

&#x20;

!\[\[15\_BountyHunter\_image0010.png]]

So there was a test account on the portal and this seems to point us towards a hashed password.

&#x20;

All of the codes seemed to be difficult to read except for bountylog.php, which housed an interesting extension.

&#x20;

!\[\[15\_BountyHunter\_image0011.png]]

&#x20;

So it seems that the website has been sending POST requests to this other PHP extension here.

&#x20;

So this is the website that is taking our XXE code and showing us the value behind it.

&#x20;

From here, we can begin to craft our own XXE payloads. This works because the data sent to the tracker.php is not being validated by the val() function in the bountySubmit function. Hence, we should be able to test it and dump out interesting things from this.

&#x20;

!\[\[15\_BountyHunter\_image0012.png]]

I base64 encoded this and then URL encoded this, and this gave me the /etc/passwd file in the output.

&#x20;

!\[\[15\_BountyHunter\_image0013.png]]

&#x20;

Now that we can read the files, there was one file I was interested in before, which was the db.php file we saw earlier.

&#x20;

That was something interesting.

Same thing as before, I used this trick to get the db.php file out in base64 encoded manner.

&#x20;

!\[\[15\_BountyHunter\_image0014.png]]

From this, base64 encode it and URL encode it, and it should output this.

!\[\[15\_BountyHunter\_image0015.png]]

&#x20;

From here, we can some mysql credentials.

!\[\[15\_BountyHunter\_image0016.png]]

&#x20;

Looking at the /etc/passwd file, there was one user there by the name of development.

!\[\[15\_BountyHunter\_image0017.png]]

Using this password, we can SSH in and grab the user flag.

!\[\[15\_BountyHunter\_image0018.png]]

Checking sudo privileges, we see this.

&#x20;

!\[\[15\_BountyHunter\_image0019.png]]

&#x20;

We can execute this one script here.

!\[\[15\_BountyHunter\_image0020.png]]

The whole thing is pretty innocent, except for this one eval function here.

!\[\[15\_BountyHunter\_image0021.png]]

This is a red flag, because from this, we can inject some arbitrary python code into this and call a shell.

This does not work very w

!\[\[15\_BountyHunter\_image0022.png]]

This is one method of doing so. Basically, we need to create a malicious ticket that has a shell within it that would generate the root shell for us through the eval function.

&#x20;

!\[\[15\_BountyHunter\_image0023.png]]

&#x20;

Here are the conditions for a good ticket.

!\[\[15\_BountyHunter\_image0024.png]]

&#x20;

Here is one failed ticket that is deemed invalid found in the same directory as the script. We can just edit this to what we want. Basically, we would need for the ticket to keep the first 3 lines, and then inject our code into the \*\* portion there, which is replaced and evaluated accordingly.

&#x20;

The first number has to be a number that gives a remainder of 4 when divided by 7. This would be the condition for it to execute the rest of the code. Afterwards, the sum that is being validated has to be more than 100 for the code to finish. I wanted to test and see if I could get a valid ticket before trying any injections.

&#x20;

!\[\[15\_BountyHunter\_image0025.png]]

This right here is a valid ticket.

&#x20;

Now, I wanted to see if we could append any form of shell that would also be executed within the eval function.

In this case, this would rely on some flaws in the eval script. Basically, the function is able to be injected with some comparison stuffs.

&#x20;

So in this case, we need to do some dynamic code injection. This can be done using some form of comparison conditions, such as true or false.

&#x20;

Say for a minute here we are interested in making the eval function execute both conditions and stuff, we would need some form of logical operator such as **and** that can cause it to skip the first portion of the script and move on to the second portion.

!\[\[15\_BountyHunter\_image0026.png]]

This is an example of a true condition, no matter what.

&#x20;

The function would evaluate part A and move on to part B, which can be our shell.

&#x20;

!\[\[15\_BountyHunter\_image0027.png]]

Using this ticket, we can get a root shell.

!\[\[15\_BountyHunter\_image0028.png]]

&#x20;

Finished.

&#x20;

&#x20;
