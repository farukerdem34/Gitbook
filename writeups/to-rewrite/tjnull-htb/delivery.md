# Delivery

Delivery

Monday, 21 February 2022

3:42 pm

This is a Linux machine.

The target IP is 10.10.10.222.

My IP is 10.10.16.9.

&#x20;

This box is made by Ippsec, so it should be good!

&#x20;

Ran a quick scan for all the ports to check on what's running.

!\[\[49\_Delivery\_image001.png]]

&#x20;

8065? Hmm. Seems to be an unassigned port.

&#x20;

Visiting the web page, we see this.

!\[\[49\_Delivery\_image002.png]]

&#x20;

Nice looking web page.

Ran a directory enumeration to check on what is present on the web page.

&#x20;

!\[\[49\_Delivery\_image003.png]]

There's a contact us page on the website, clicking it reveals this.

&#x20;

!\[\[49\_Delivery\_image004.png]]

&#x20;

Clicking it leads to port 8065. I'll go through the other website later.

&#x20;

!\[\[49\_Delivery\_image005.png]]

I created an account first.

!\[\[49\_Delivery\_image006.png]]

Then logged in to see what's going on.

We need to resend some email to log in.

&#x20;

I intercepted the response in Burpsuite, which does not really seem interesting anyways.

&#x20;

!\[\[49\_Delivery\_image007.png]]

There was another website that was present, called helpdesk.

&#x20;

Let's take a look at it.

!\[\[49\_Delivery\_image008.png]]

&#x20;

There's this huge page here, and we can check ticket statuses. I created a ticket, and was given this email address.

!\[\[49\_Delivery\_image009.png]]

So basically, this is like an email client to me.

I created another account with this email, as shown here.

&#x20;

It says I can add more information to my ticket, hence I can simply use that email and check the status of the ticket to successfully get the verification mail.

&#x20;

I created another ticket. I then used this same email to then sign up for an account.

&#x20;

From here, I was able to view my ticket.

!\[\[49\_Delivery\_image0010.png]]

&#x20;

Logging in presented two things:

!\[\[49\_Delivery\_image0011.png]]

&#x20;

So there are SSH credentials and also a password to the hashes.

SSH in.

{width="8.3125in" height="4.40625in"}

Afterwards I installed linpeas.sh, just to check on what is going on. Didn't find much though.

Anyways I headed into /opt, and this had a file here, which is unusual.

&#x20;

I took a look at the config files and found a config.json.

!\[\[49\_Delivery\_image0013.png]]

&#x20;

There were a few hashes within this.

&#x20;

!\[\[49\_Delivery\_image0014.png]]

&#x20;

Most importantly, there was a database password and stuff within this

&#x20;

!\[\[49\_Delivery\_image0015.png]]

We should definitely look into this.

First I check netstat, on whether it was alive.

&#x20;

!\[\[49\_Delivery\_image0016.png]]

&#x20;

Good enough for me.

{width="10.197916666666666in" height="2.9479166666666665in"}

&#x20;

Log in and let's see what we can find.

{width="4.125in" height="2.4791666666666665in"}

&#x20;

Proceeding with this, let's see the database and what's inside.

&#x20;

{width="4.0625in" height="1.6458333333333333in"}

&#x20;

!\[\[49\_Delivery\_image0020.png]]

&#x20;

I saw Users, I selected everything I could get.

!\[\[49\_Delivery\_image0021.png]]

&#x20;

There seems to be some form of hash here.

This is clearly the password, and we need to crack this hash.

&#x20;

I have a password for this hash, so let's just try it using hashcat.

Based on the file format, this is mode 3200. This is not a dictionary attack, but rather a rule attack that would allow for us to break into the machine.

The resultant password is Password!21.

&#x20;

This would allow us to su into root and grab the flag.

!\[\[49\_Delivery\_image0022.png]]

&#x20;
