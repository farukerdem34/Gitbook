# Mango

Mango

Thursday, 10 February 2022

9:00 pm

This is a Linux machine.

My IP is 10.10.16.9.

The target IP is 10.10.10.162.

&#x20;

Enumeration time.

Fast scan shows that SSH, HTTP and HTTPS are open.

&#x20;

Detailed scan shows this.

!\[\[32\_Mango\_image001.png]]

&#x20;

Directory enumeration underway.

Port 80, seems boring.

&#x20;

!\[\[32\_Mango\_image002.png]]

&#x20;

HTTPS reports something different.

&#x20;

!\[\[32\_Mango\_image003.png]]

&#x20;

&#x20;

There's a searchbox.

There's also a analytics.php file, which is more interesting to me.

&#x20;

!\[\[32\_Mango\_image004.png]]

&#x20;

!\[\[32\_Mango\_image005.png]]

&#x20;

A quick check on the engine used, and it is indeed flexmonster.

&#x20;

!\[\[32\_Mango\_image006.png]]

Interestingly, we are logged in as MrR3boot.

!\[\[32\_Mango\_image007.png]]

&#x20;

At this point, I was at an end. However, I did find one little clue. Viewing the certificate of the website.

&#x20;

!\[\[32\_Mango\_image008.png]]

&#x20;

Interesting. When we visit the site, we have a log in page!

&#x20;

!\[\[32\_Mango\_image009.png]]

SQLmap reveals nothing for me. That forget password button is useless too.

&#x20;

I know the user is the one I found before, randomly. I looked at the reviews of the website and it reveals that this uses NoSQL.

&#x20;

Quickly downloaded the nosqli tool from kalilunuxtutorials.com.

Got it running alive and well.

While waiting, I'm gonna read up on NoSQL injection.

&#x20;

So the basis of this is that this targets databases, like Redis and MongoDB that DO NOT use SQL language, rendering SQLi useless.

For example, the normal query to escape from a SQL injection would be '

&#x20;

IN other DB, they use things like $eq, $ne etc. A bit like assembly.

$query = array("user" => $\_POST\["username"], "password" =>\
$\_POST\["password"]);

This for example is one possible payload.

&#x20;

Clearly this uses MongoDB. Play on words from Mango.

&#x20;

!\[\[32\_Mango\_image0010.png]]

&#x20;

This works out well.

&#x20;

So we found out that this weakness exists. This application I'm using however, does not supprot any form of password enumeration or whatever. So for now, we have to rely on other tools.

[https://github.com/an0nlk/Nosql-MongoDB-injection-username-password-enumeration](https://github.com/an0nlk/Nosql-MongoDB-injection-username-password-enumeration)

&#x20;

I used this to enumerate out my password and username easily.

Used it to enumerate out passwords and usernames.

&#x20;

!\[\[32\_Mango\_image0011.png]]

&#x20;

!\[\[32\_Mango\_image0012.png]]

Verified that there is indeed an admin account.

&#x20;

Now time to enumerate the passwords.

I don't know which password which password.

&#x20;

!\[\[32\_Mango\_image0013.png]]

The second password is the one that works for admin.

&#x20;

!\[\[32\_Mango\_image0014.png]]

But I don't think this actually works properly.

Both accounts show this page. So there's an email basically. Let's try SSH.

&#x20;

I SSHed in using mango and the admin's password!

&#x20;

!\[\[32\_Mango\_image0015.png]]

So we are the lowest privilege available on the machine. The user flag is in the admin account, but we are unable to access that.

Ported over LinEnum.sh because I can LOL.

&#x20;

Let's see what I can do on this machine.

Here are the interesting folders.

&#x20;

{width="9.072916666666666in" height="0.8541666666666666in"}

&#x20;

!\[\[32\_Mango\_image0017.png]]

Headed onto GTFObins, and we can see that JJS is an SUID that can get us a reverse shell. In this case, it seems like it will get us the admin account shell.

&#x20;

Let's try it.

&#x20;

!\[\[32\_Mango\_image0018.png]]

&#x20;

Seems that we just aren't allowed to execute this file.

&#x20;

I went so far ahead of myself I forgot that I can just try the other password...

!\[\[32\_Mango\_image0019.png]]

Cool, anyway we're already here, so let's try a root shell.

However much I tried, this did not work out for me.

I referred to walkthroughs, as I was sure this was the way to go.

&#x20;

Realised that the -p flag was the one that was missing.

&#x20;

{width="12.739583333333334in" height="0.6041666666666666in"}

&#x20;

!\[\[32\_Mango\_image0021.png]]

Pretty fun box, I enjoyed learning about the SQL injection part.

&#x20;

**\[Other methods that work]{.underline}**

Additionally, there were other methods I could have used instead.

&#x20;

!\[\[32\_Mango\_image0022.png]]

I could have appended my private key into the root's authorized\_keys file, which would have worked as well.

&#x20;

This would have allowed me to SSH.

&#x20;

Other methods include that I can just add myself to sudo, and allow me to simply add it in, we can append this code.

!\[\[32\_Mango\_image0023.png]]

This would effectively add in our user account into the sudoers file, and allow us to sudo and change the users as we wish.

&#x20;

!\[\[32\_Mango\_image0024.png]]

This would have been a much cleaner execution.

&#x20;

What I learnt:

1. NoSQL injections exist!
2. The usage of an executable script would allow for me to append my private key into the code, keep this in mind when something does not work.
3. Always check all passwords.
4. The ability to run one script as any account, gives us so many ways of exploitation. Take note of the SSH and the appending of our user to sudo! Those would come in handy some day.

&#x20;

&#x20;
