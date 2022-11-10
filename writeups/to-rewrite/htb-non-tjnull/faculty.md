# Faculty

Faculty

Tuesday, July 5, 2022

10:02 AM

Nmap scan:

!\[\[35\_Faculty\_image001.png]]

&#x20;

Port 80:

!\[\[35\_Faculty\_image002.png]]

&#x20;

Within this page, there is also an administrator login.

!\[\[35\_Faculty\_image003.png]]

&#x20;

When trying some form of SQL Injections, we get this:

!\[\[35\_Faculty\_image004.png]]

&#x20;

I think this is an SQL error. This looks like one.

&#x20;

We can Sqlmap this with level 5 and risk 3. Afterwards, we can try to inject some kind of query into this.

&#x20;

{width="9.822916666666666in" height="0.65625in"}

&#x20;

So we know we have SQL Injection within this area, perhaps we can write a PHP Shell or something.

!\[\[35\_Faculty\_image006.png]]

&#x20;

Now we can dump the database.

{width="8.916666666666666in" height="3.09375in"}

&#x20;

There's this one scheduling\_db with a faculty one, perhaps this would have the ID that we need.

{width="3.6041666666666665in" height="2.2604166666666665in"}

&#x20;

{width="7.0in" height="0.8125in"}

&#x20;

{width="8.21875in" height="1.8958333333333333in"}

&#x20;

This hash cannot be cracked.

We can proceed to dump out the faculty table to try and see if we can get any form of faculty number.

&#x20;

We can get the Pin Number from faculty and then login as the administrator.

&#x20;

&#x20;

!\[\[35\_Faculty\_image0011.png]]

&#x20;

Turns out we also could have just used some form of SQL Injection to get us in.

&#x20;

We can view the other users present.

!\[\[35\_Faculty\_image0012.png]]

&#x20;

Within the page, there seems to be this PDF generator.

!\[\[35\_Faculty\_image0013.png]]

&#x20;

When viewed, we can see that this uses mPDF.

!\[\[35\_Faculty\_image0014.png]]

&#x20;

This thing has some form of RFI for us. And we can make use of this.

&#x20;

From this, we can view the requests in Burpsuite.

!\[\[35\_Faculty\_image0015.png]]

&#x20;

There would be a POST request to /admin/download.php, which would have a base64 and double URL encoded HTML text thing.

&#x20;

!\[\[35\_Faculty\_image0016.png]]

&#x20;

This would basically mean that we are able to include our own sort of code to be able to read some files perhaps.

&#x20;

We can also download the PDF and then exiftool it to find out more about the application.

{width="7.09375in" height="4.0625in"}

&#x20;

I was able to find this page which is good.

[https://medium.com/@jonathanbouman/local-file-inclusion-at-ikea-com-e695ed64d82f](https://medium.com/@jonathanbouman/local-file-inclusion-at-ikea-com-e695ed64d82f)

&#x20;

This explains how the PoC works, and we can use it to read files.

!\[\[35\_Faculty\_image0018.png]]

&#x20;

We can see that it sort of works in trying to access the file.

&#x20;

However, it does not attach the file.\
In this case, we can try to remove all the escape characters and try again.

&#x20;

!\[\[35\_Faculty\_image0019.png]]

&#x20;

!\[\[35\_Faculty\_image0020.png]]

&#x20;

Success! We can basically include attached files to this.

&#x20;

We can view the possible users.

{width="5.65625in" height="1.375in"}

.

!\[\[35\_Faculty\_image0022.png]]

&#x20;

Then double url encode and base64 encode this.

!\[\[35\_Faculty\_image0023.png]]

&#x20;

Hmm, this user does not have any SSH keys.

In this case, we can try to read some config files.

We can read the first file of which there was mention of the admin\_class.php, and then be able to view the connections made.

!\[\[35\_Faculty\_image0024.png]]

&#x20;

Db\_connect.php could be a possible thing.

!\[\[35\_Faculty\_image0025.png]]

&#x20;

I'm not sure if this password is reused or something.

&#x20;

I tried SSH with gbyolo and it worked!

{width="2.9375in" height="0.5in"}

&#x20;

!\[\[35\_Faculty\_image0027.png]]

&#x20;

We have mail?

!\[\[35\_Faculty\_image0028.png]]

&#x20;

It seems that also, we may run meta-git as the developer.

{width="7.802083333333333in" height="1.5in"}

&#x20;

I'm going to try and see what I can do to this, maybe I was spawn a shell, or read files again.

&#x20;

!\[\[35\_Faculty\_image0030.png]]

&#x20;

There's this clone one.

I found some exploits that allowed for RCE.

[https://hackerone.com/reports/728040](https://hackerone.com/reports/728040)

!\[\[35\_Faculty\_image0031.png]]

&#x20;

This was the PoC, and we could certainly try to read the private key of the user.

!\[\[35\_Faculty\_image0032.png]]

&#x20;

Cool.

{width="4.40625in" height="1.15625in"}

&#x20;

Flag:

!\[\[35\_Faculty\_image0034.png]]

&#x20;

PE:

I ran a LinPEAS to enuemrate for me.

There was the cronjob running, but I wasn't sure how this would function for us, and if the mail is even sent by root or developer.

&#x20;

!\[\[35\_Faculty\_image0035.png]]

&#x20;

Extremely unlikely for this cronjob to be the exploit, too easy for this machine.

&#x20;

Through some enumeration, we can find that developer is part of the debug group.

!\[\[35\_Faculty\_image0036.png]]

&#x20;

Interestingly, this debug group is able to run gdb within the machine.

{width="5.927083333333333in" height="0.65625in"}

&#x20;

So for whatever reason, we have the ability to debug stuff.

GDB also has the ability to attach itself to a process. Perhaps we could attach ourselves to a process that is running as root?

&#x20;

I found out that we were also able to spawn child processes, meaning we could potentially attach to a process, and from there be able to spawn processes as the root user, as we are attached to a root process.

&#x20;

We can start by finding processes that are running as root.

!\[\[35\_Faculty\_image0038.png]]

&#x20;

!\[\[35\_Faculty\_image0039.png]]

&#x20;

I used this one.

!\[\[35\_Faculty\_image0040.png]]

&#x20;

We can use the -p option to attach ourselves there and then be able to spawn in processes using the call function.

!\[\[35\_Faculty\_image0041.png]]

&#x20;

{width="8.947916666666666in" height="0.6875in"}

&#x20;

{width="6.53125in" height="1.8645833333333333in"}

&#x20;

Flag:

!\[\[35\_Faculty\_image0044.png]]

81e6b242b4d72cccdc6fab899a54bd9a

&#x20;

Why does this work?

Well, because as the debug user, we can basically attach ourselves to any root process, and be able to debug that. We can stop the execution process and make that one root thread execute or spawn whatever other functions that we want. This would let us basically execute commands as root through system() function.

&#x20;
