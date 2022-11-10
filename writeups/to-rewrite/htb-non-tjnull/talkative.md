# Talkative

Talkative

Saturday, June 11, 2022

3:05 PM

Nmap scan:

!\[\[22\_Talkative\_image001.png]]

&#x20;

!\[\[22\_Talkative\_image002.png]]

&#x20;

Port 80:

!\[\[22\_Talkative\_image003.png]]

&#x20;

Add to hosts file:

!\[\[22\_Talkative\_image004.png]]

&#x20;

!\[\[22\_Talkative\_image005.png]]

&#x20;

Team:

!\[\[22\_Talkative\_image006.png]]

&#x20;

Some users present.

&#x20;

Port 8080:

!\[\[22\_Talkative\_image007.png]]

&#x20;

So there's an exploit for this out.

Anyways, there is an R module there.

!\[\[22\_Talkative\_image008.png]]

&#x20;

{width="8.75in" height="2.7604166666666665in"}

&#x20;

We can actually execute commands as this.

But there's limited extent to what we can do.

{width="12.25in" height="2.4479166666666665in"}

&#x20;

Cool.

There is a method of which we can use to perhaps adds argumetns to this.

{width="10.71875in" height="3.78125in"}

&#x20;

{width="12.166666666666666in" height="3.6354166666666665in"}

&#x20;

In /root, there's a password there.

{width="9.979166666666666in" height="1.75in"}

&#x20;

Looks like a bunch of nonsense, so let's try to find out more about omv files.

{width="12.854166666666666in" height="6.270833333333333in"}

&#x20;

We can gain a reverse shell though.

!\[\[22\_Talkative\_image0015.png]]

&#x20;

{width="7.96875in" height="1.6875in"}

&#x20;

Then, take the base64 of the omv file and we can transfer it back to our machine.

!\[\[22\_Talkative\_image0017.png]]

&#x20;

!\[\[22\_Talkative\_image0018.png]]

&#x20;

{width="3.3229166666666665in" height="1.84375in"}

&#x20;

There are some passwords.

{width="9.791666666666666in" height="1.2083333333333333in"}

&#x20;

Right, we have creds, but I'm not sure where to use this.

&#x20;

Gobuster scans:

!\[\[22\_Talkative\_image0021.png]]

&#x20;

This does not work well, because it seems to have anti brute-force stuff.

&#x20;

I know bolt-administration is something to do with a login or something.

Bolt is also a CMS name, so I guessed /bolt, and it kinda worked.

We have some passwords and some usernames, but anything to do with SSH is not possible as it's filtered.

&#x20;

/bolt:

!\[\[22\_Talkative\_image0022.png]]

&#x20;

!\[\[22\_Talkative\_image0023.png]]

&#x20;

We can try the credentials for all of them that we had found. This one worked.

!\[\[22\_Talkative\_image0024.png]]

&#x20;

Under file management and theme editor, we can see how there are some files that are executed with .twig extensions.

!\[\[22\_Talkative\_image0025.png]]

&#x20;

This means we can use this for some SSTI.

!\[\[22\_Talkative\_image0026.png]]

&#x20;

Select base-2021 and then we can use whatever template we want.

!\[\[22\_Talkative\_image0027.png]]

&#x20;

!\[\[22\_Talkative\_image0028.png]]

&#x20;

!\[\[22\_Talkative\_image0029.png]]

&#x20;

Cool, we have some SSTI through the use of the twig templates. Now we need to think about how to execute some arbitrary code using this template.

| We can try to execute arbitrary commands using this to confirm it. |
| ------------------------------------------------------------------ |
|                                                                    |

!\[\[22\_Talkative\_image0031.png]]

&#x20;

From this, we can see that there is only root present on this machine.

So from here, we would need to somehow gain a reverse shell from this.

This payload works.

{width="10.78125in" height="0.8541666666666666in"}

&#x20;

{width="8.010416666666666in" height="2.15625in"}

&#x20;

We can spawn an interactive shell with script -qc.

TTY Shell:

!\[\[22\_Talkative\_image0034.png]]

&#x20;

We still have valid credentials from earlier however, so I thought of using SSH to get into another host.

!\[\[22\_Talkative\_image0035.png]]

&#x20;

{width="7.708333333333333in" height="2.0729166666666665in"}

&#x20;

!\[\[22\_Talkative\_image0037.png]]

&#x20;

PE2:

Ran Linpeas:

!\[\[22\_Talkative\_image0038.png]]

&#x20;

There are a lot of processes being run by root, such as all the different containers each web machine is in.

&#x20;

Pspy:

!\[\[22\_Talkative\_image0039.png]]

&#x20;

!\[\[22\_Talkative\_image0040.png]]

&#x20;

One unquoted path here.

!\[\[22\_Talkative\_image0041.png]]

&#x20;

Then we can see this one bit here. So there's a script that's running, and it seems to be updating mongodb every so often. Of course we can't read the script, but we can see what's its doing roughly.

&#x20;

We don't have write privileges over anything regarding the system PATH.

WIP as this is mad hard...

&#x20;
