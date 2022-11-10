# PostFish

PostFish

Tuesday, 22 March 2022

10:02 pm

This is a Linux machine with these ports:

&#x20;

!\[\[32\_PostFish\_image001.png]]

&#x20;

Wanted to test what kind of usernames there were on this SMTP server, seeing as the desc of the box was to check our emails.

&#x20;

The webserver had these 4 users on them:

!\[\[32\_PostFish\_image002.png]]

Good to know.

&#x20;

I ran smtp-user-enum to determine more emails that exist on the web server.

&#x20;

Interestingly, I found a HR email, which is something that is related to the users above. This could be a sign that I had to use their names for something.

&#x20;

!\[\[32\_PostFish\_image003.png]]

I tried a few more usernames like SALES, IT and LEGAL.

&#x20;

!\[\[32\_PostFish\_image004.png]]

&#x20;

!\[\[32\_PostFish\_image005.png]]

Seems like all of these users have some form of emails on the web server.

&#x20;

From here, we can perhaps begin pentesting and reading into some emails. I doubt that we would need to send phishing emails over here.

Went trying random stuff with hr:hr, and postfish:postfish.

&#x20;

Eventually, sales:sales worked!

!\[\[32\_PostFish\_image006.png]]

There was one message within this.

&#x20;

{width="11.40625in" height="2.1979166666666665in"}

Good to know that these links would be sent out somewhere. Perhaps from here, we can craft some phishing email to make it look like a password link.

&#x20;

Made one fast email here.

!\[\[32\_PostFish\_image008.png]]

This does not seem to work...This would mean the email has to come from someone else that is within IT and is not IT@postfish.off.

&#x20;

Looking at the recipient of the message, it seems this could also be off.

The name on the website is Bryan Moore, which is probably someone we need to use.

&#x20;

So we know that some user from sales is to receive this mail, and that we need to send it from the it@postfish.off.

&#x20;

From here, we can use the ability to create a nc listener (similar to SneakyMailer on HTB!) and listen for stuff. We just need to send it to the right user.

!\[\[32\_PostFish\_image009.png]]

This seems to be a good user to use.

&#x20;

From there, we just send another mail with him in it. Then on our NC listener, we get a password!

!\[\[32\_PostFish\_image0010.png]]

&#x20;

Using this password, we can SSH into the machine!

!\[\[32\_PostFish\_image0011.png]]

&#x20;

From here, I got linpeas onto the machine and began enumerating about how to go about this.

!\[\[32\_PostFish\_image0012.png]]

&#x20;

This was something new.

Well, disclaimer is something that would append some stuff to the end of a mail should it be from some mail from another user.

!\[\[32\_PostFish\_image0013.png]]

We can totally use this to get a shell or something back.

We can edit this disclaimer script, which is pretty cool I guess.

&#x20;

Basically, this script would execute should the user receive a mail or something I think.

&#x20;

Let's just append some bash script to the script.

!\[\[32\_PostFish\_image0014.png]]

&#x20;

The first time I did it, it seems that the script renews itself and I do not get a shell back...

And so I did this, and got a shell back as filter!

!\[\[32\_PostFish\_image0015.png]]

The rest is easy.

&#x20;

!\[\[32\_PostFish\_image0016.png]]

&#x20;

!\[\[32\_PostFish\_image0017.png]]

&#x20;

!\[\[32\_PostFish\_image0018.png]]

&#x20;

!\[\[32\_PostFish\_image0019.png]]

&#x20;

I admit, definitely referred to a walkthrough of another box from HTB for this disclaimer portion, as I was unsure of what to do.

&#x20;

But still, learnt a lot from this, especially with email crafting and SMTP forcing.

&#x20;
