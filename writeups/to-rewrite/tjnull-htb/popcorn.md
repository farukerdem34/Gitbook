# Popcorn

Popcorn

Tuesday, 8 February 2022

10:29 pm

This is a Linux machine.

My IP is 10.10.16.5.

The target IP is 10.10.10.6.

&#x20;

Enumeration.

&#x20;

!\[\[27\_Popcorn\_image001.png]]

&#x20;

Do a directory enumeration as well.

This is the web page shown on port 80.

!\[\[27\_Popcorn\_image002.png]]

This is the result of the directory enumeration. (Ignore the errors <3)

&#x20;

!\[\[27\_Popcorn\_image003.png]]

This is running a PHP server, might as well check with PHP filenames as well as common.txt.

&#x20;

Test.php and /test go to the same page, of which it's just the information of the server and stuff. Take note of the system, which is Linux Popcorn 2.6.31-14-generic-pae. Never know when that could come in handy.

&#x20;

&#x20;

!\[\[27\_Popcorn\_image004.png]]

&#x20;

/torrent reveals that this is using Torrent Hoster, and on it there are username and password fields. Not sure what to do with that for now.

!\[\[27\_Popcorn\_image005.png]]

&#x20;

&#x20;

/rename reveals that the index.php is actually able to rename files and stuff.

!\[\[27\_Popcorn\_image006.png]]

&#x20;

I'm not sure what this can be used for, but we just keep note of it for now.

&#x20;

On the bottom of the page, it seems they are using 2007 version of TorrentHoster, which is bound to have some form of vulnerabilities.

&#x20;

I created an account to see what I could do. There's this upload portion of the code, where it seems that we can upload anything.

!\[\[27\_Popcorn\_image007.png]]

&#x20;

Referring to searchsploit, there is a vulnerability that would allow us to upload shells and stuff to the machine.

{width="3.8854166666666665in" height="2.1041666666666665in"}

&#x20;

There seems to be an upload made by the Admin as well, so let's take a look at that.

!\[\[27\_Popcorn\_image009.png]]

&#x20;

Does not seem interesting in the slightest.

&#x20;

Anyways it seems that all of our different files are being rejected. There is a WAF to check whether the file is a valid torrent file. Let's see if we can bypass that somehow.

&#x20;

Well I was completely unable to bypass the WAF by my own means, so I realised that the admin uploaded some kind of Kali torrent. I downloaded a torrent from the Kali downloads page, and after uploading it brought me to here.

&#x20;

&#x20;

!\[\[27\_Popcorn\_image0010.png]]

&#x20;

I could edit this torrent. Let's check that out.

!\[\[27\_Popcorn\_image0011.png]]

Seems like we are allowed to upload screenshots. Back to trying random stuff! I changed the content-type and it got uploaded!

!\[\[27\_Popcorn\_image0012.png]]

&#x20;

Now we just need to check the /torrent/uploads folder and execute our reverse shell. Bam.

&#x20;

!\[\[27\_Popcorn\_image0013.png]]

&#x20;

!\[\[27\_Popcorn\_image0014.png]]

&#x20;

We seem to spawn in as www-data.

!\[\[27\_Popcorn\_image0015.png]]

&#x20;

All we gotta do is just privilege escalate for now.

Ported over LinEnum.sh!

&#x20;

Checked through the scan and determined some cool stuff.

{width="8.166666666666666in" height="2.7291666666666665in"}

&#x20;

!\[\[27\_Popcorn\_image0017.png]]

&#x20;

!\[\[27\_Popcorn\_image0018.png]]

&#x20;

While we're here, we can grab that user.txt file.

At this point I was stuck again, referred to a walkthrough because I simply did not know what to do.

&#x20;

In this case, I took note of the fact that there was a .cache folder within the user file. This had a MOTD folder within it, which is vulnerable to some exploits.

!\[\[27\_Popcorn\_image0019.png]]

&#x20;

This basically had a script that would automatically get us the root.shell. How it runs is this:

1. Generates a key pair within a .ssh directory.
2. Copies the public key into authorized\_keys in the server and sets the permissions.
3. When we take this private key and use it for SSH, we are able to generate the .cache file that was originally present in only the user's file.
4. From here, we are able to remove the .cache file created.
5. We then create a symbolic link between that cache file and the /etc/passwd.
6. This would cause the .cache file to point towards /etc/passwd, which is owned by www-data.
7.  In short, this creates a file that is able to write directly to /etc/passwd, of which case we can simply just add our own user.

    a. **This would be done using openssl, and appending this straight to the passwd file.**

    b. **Openssl passwd -1 username**

```
<!-- -->
```

1. By owning the /etc/passwd, we can create more root users! We are also able to set the home directory to whatever we want, in this case being the root directory in order to fully pwn the machine.

&#x20;

This whole exploit is based on the fact that we are able to generate our own .cache file which is basically owned by us. The script shown in MSF would automate this task.

&#x20;

Still, really cool way of using the ssh functions and the fact that the SSH port is open on the machine.

&#x20;

At the current level I'm at, I will not be going into how the guy found this vulnerability, but rather just understanding how to use it and stuff.

&#x20;

Overall, this was a good box, I learnt a lot.

&#x20;

I ran the script myself, and I was basically able to do whatever was explained.

&#x20;

&#x20;

> &#x20;
