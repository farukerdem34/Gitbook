# Book

Book

Wednesday, 9 March 2022

2:16 pm

This is a Linux machine.

The target IP is 10.10.10.176.

My IP is 10.10.16.9

&#x20;

Enumeration time.

!\[\[73\_Book\_image001.png]]

&#x20;

Website:

!\[\[73\_Book\_image002.png]]

&#x20;

Ran a directory scan anyways:

!\[\[73\_Book\_image003.png]]

Admin panel huh?

&#x20;

Let's try it.

!\[\[73\_Book\_image004.png]]

Same thing.

&#x20;

Let's view the source code of both pages. The admin page was not really interesting, but the regular login page was:

{width="6.3125in" height="2.125in"}

This little snippet here validates the number of characters that we can key in.

A bit of googling led me to SQL Truncation attack, which can be used to truncate certain queries that are made.

&#x20;

I noticed there's this signup button for us, and we can key in an email.

&#x20;

From here, based on SQL Truncation, as long as our username and email that we give bypasses these limits, we can basically create a username with admin and the email be admin@book.htb, which is likely the administrator of this website.

&#x20;

Let's use Burpsuite to generate this.

&#x20;

Firstly, I verified the existence of this administrator account:

!\[\[73\_Book\_image006.png]]

Then, we can generate our user for the admin panel.

!\[\[73\_Book\_image007.png]]

This works because the max character count is as such, and anything else would be truncated. We just need to hit the maximum character count and then its considered 'a different username'.

&#x20;

After sending this request, we can login to the admin panel using the email and the password given.

!\[\[73\_Book\_image008.png]]

&#x20;

There were a few tabs to see, and collections generates PDF files or something that take a while.

!\[\[73\_Book\_image009.png]]

Let's try to view the PDF files.

It has nothing of interest.

&#x20;

From here, I knew that the PDF was taking too long to generate and that it was getting this file was somewhere.

&#x20;

I created a non-admin account and noticed that I was allowed to upload collections.

!\[\[73\_Book\_image0010.png]]

Through this, perhaps there was a way to make it such that I was able to view code.

I didn't know what I could do, hence I resorted to a walkthrough.

Turns out we are able to use stored XSS to inject scripts into both the book title and the author:

&#x20;

\<script>x\*\*=new**XMLHttpRequest;x.onload**=function\*\*(){document.write(**this**.responseText)};x.open("GET","file:///etc/passwd");x.send();\</script>

&#x20;

_From <_[_https://0xdf.gitlab.io/2020/07/11/htb-book.html#xss-file-read_](https://0xdf.gitlab.io/2020/07/11/htb-book.html#xss-file-read)_>_

&#x20;

I uploaded this and downloaded the collections pdf, which resulted in me getting the /etc/passwd file. From there I saw that the user was called reader.

!\[\[73\_Book\_image0011.png]]

&#x20;

I could change this and start reading the private key of the user reader

Change rthe directory from the /etc/passwd file to the /home/reader/.ssh/id\_rsa, and from there we should the private key.

&#x20;

Input this as the upload:

\<script> x=new XMLHttpRequest; x.onload=function(){ document.write(this.responseText) }; x.open("GET","file:///home/reader/.ssh/id\_rsa"); x.send(); \</script>

&#x20;

Afterwards, we should get the user "reader" private key.

&#x20;

!\[\[73\_Book\_image0012.png]]

This would get us here.

&#x20;

!\[\[73\_Book\_image0013.png]]

There's this backups directory, and when read, one contains nothing and the other contains this.

!\[\[73\_Book\_image0014.png]]

&#x20;

Based on the timestamp there, it seems that there either could be other hosts on this network, or that there's a cronjob going on.

The IP address does not reveal anything of interest.

&#x20;

I ran a linpeas to check on everything, as this file seems to be written by someone else.

I also ran a quick pspy to analyse the different things being run here. I noticed this:

!\[\[73\_Book\_image0015.png]]

This was beng run by root.

Googled for some logrotate weaknesses based on its version and found some:

!\[\[73\_Book\_image0016.png]]

&#x20;

&#x20;

!\[\[73\_Book\_image0017.png]]

[This](https://github.com/whotwagner/logrotten/blob/master/logrotten.c) Github had the exploit, and all we needed to do was compile the code given and run it basically.

&#x20;

I compiled the code and ran the exploit as per how it is done, but it did not work the first time, and I realised I forgot to put the \ for the id command within payload file.

&#x20;

I echoed some random string into access.log, and it did this:

!\[\[73\_Book\_image0018.png]]

Now, it seems to have written something somewhere.

&#x20;

I tried again. This time it worked with a python shell! However, I can tell this method only works for like 10 seconds. This would work with the python shell that we can get from pentestmonkey.

!\[\[73\_Book\_image0019.png]]

&#x20;

!\[\[73\_Book\_image0020.png]]

Tried again to quickly grab the flag!

You can see me fail up there, as the shell ended then.

&#x20;

I could totally grab the id\_rsa, but seeing as the shell exists for 10 seconds, nah.

&#x20;

&#x20;
