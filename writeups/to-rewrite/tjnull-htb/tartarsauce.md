# TartarSauce

TartarSauce

Wednesday, 23 February 2022

1:58 pm

This is a Linux machine.

The target IP is 10.10.10.88.

My IP is 10.10.16.9.

&#x20;

Nmap scans reveal one singular port that is open.

!\[\[53\_TartarSauce\_image001.png]]

Cool looking page though.

!\[\[53\_TartarSauce\_image002.png]]

&#x20;

Analysis of this text image does not reveal much. I ran a vuln script to check if it was vulnerable in anyway. In the mean time, I checked out directories that were present on the website, starting with robots.txt.

!\[\[53\_TartarSauce\_image003.png]]

Indeed Nmap reveals that there is one.

The first one revealed nothing, the second one revealed that there was a monstra 3.0.4 web page.

&#x20;

!\[\[53\_TartarSauce\_image004.png]]

A searchsploit reveals there an RCE for this version.

{width="8.510416666666666in" height="2.8854166666666665in"}

&#x20;

However, we need to be authenticated. All other directories do not exist.

&#x20;

I used the first guess of password to log in, which was admin:admin. Lucky guess.

&#x20;

!\[\[53\_TartarSauce\_image006.png]]

Now time to exploit.

{width="11.666666666666666in" height="3.5416666666666665in"}

Looks good to me.

&#x20;

However, it turns out the administrator cannot upload stuff. Only editors can. I cannot change my privileges as admin either, I can only remain as admin.

At this point, it seems that this does nothing at all.

&#x20;

I ran a directory search on the main website to see if there are other things. I think this whole monstra thing is a troll, and this is way too easy for a medium difficulty box too.

&#x20;

That's when I found it...

!\[\[53\_TartarSauce\_image008.png]]

&#x20;

!\[\[53\_TartarSauce\_image009.png]]

Much better.

A bit of scanning reveals this.

!\[\[53\_TartarSauce\_image0010.png]]

Which would direct us to here.

!\[\[53\_TartarSauce\_image0011.png]]

Can't get much from this, hence I went back to Wpscan to determine what kind of plugins, templates and stuff.

!\[\[53\_TartarSauce\_image0012.png]]

So wpscan is limiting my scans, trying to get people to pay for their services.

I went to look up a walkthrough for the correct enumeration because I was too lazy to find another tool that scans WP so well.

&#x20;

!\[\[53\_TartarSauce\_image0013.png]]

These are the correct vulnerabilities that are used. I checked the readme for gwolle and akismet.

&#x20;

!\[\[53\_TartarSauce\_image0014.png]]

Damnit LOL.

&#x20;

So this is indeed a vulnerable version.

!\[\[53\_TartarSauce\_image0015.png]]

We have this, so let's try it.

!\[\[53\_TartarSauce\_image0016.png]]

&#x20;

TLDR, we can include a file here and we can execute arbitrary code.

Create a php payload, and this would allow us to execute code within the browser.

This does not work out with a normal web shell, so I opted for a full reverse PHP shell.

&#x20;

Works out well.

!\[\[53\_TartarSauce\_image0017.png]]

&#x20;

!\[\[53\_TartarSauce\_image0018.png]]

&#x20;

!\[\[53\_TartarSauce\_image0019.png]]

Cool! We are www-data.

There's one user called onuma.

&#x20;

!\[\[53\_TartarSauce\_image0020.png]]

Let's go to the WP directory.

&#x20;

From here, I found the wordpress password within the wp-config.php file.

&#x20;

!\[\[53\_TartarSauce\_image0021.png]]

Always good to try an su first.

Did not work.

&#x20;

So let's log into MySQL to see what I can get.

!\[\[53\_TartarSauce\_image0022.png]]

There's two databases.

!\[\[53\_TartarSauce\_image0023.png]]

&#x20;

!\[\[53\_TartarSauce\_image0024.png]]

Let's get that one password out.

!\[\[53\_TartarSauce\_image0025.png]]

Of course, its hashed.

&#x20;

Get that hash out and let's try john to decrypt it.

In the meantime, let's explore the box.

&#x20;

I found out I can execute sudo without a password.

&#x20;

!\[\[53\_TartarSauce\_image0026.png]]

So turns out I can execute /bin/tar.

&#x20;

Went to GTFO bins and used the first command.

!\[\[53\_TartarSauce\_image0027.png]]

I stopped john because I don't think that's the solution.sud

So the onuma means that I can execute this on onuma's behalf. So let's try it.

&#x20;

!\[\[53\_TartarSauce\_image0028.png]]

Cool. Grab the user flag and within the user directory, there's something else.

&#x20;

!\[\[53\_TartarSauce\_image0029.png]]

Empty file. Whatever.

&#x20;

From here, we have to PE.

I ran linpeas to see what I could find. The usual opt directory was empty :(

What's interesting was these:

&#x20;

!\[\[53\_TartarSauce\_image0030.png]]

&#x20;

!\[\[53\_TartarSauce\_image0031.png]]

Take note this machine has no SSH ports, so don't think about keys.

!\[\[53\_TartarSauce\_image0032.png]]

&#x20;

This didn't yield anything, so I got pspy into the machine.

&#x20;

!\[\[53\_TartarSauce\_image0033.png]]

Let's see what processes is root running systematically.

&#x20;

Looking at the processes, there's this one script run by root that does not look very...linuxy.

&#x20;

!\[\[53\_TartarSauce\_image0034.png]]

So this is a bash script that is running a bunch of stuff as it turns out.

&#x20;

I'll be trying my best to analyse this script here.

&#x20;

\#!/bin/bash

&#x20;

\#-------------------------------------------------------------------------------------

\# backuperer ver 1.0.2 - by ȜӎŗgͷͼȜ

\# ONUMA Dev auto backup program

\# This tool will keep our webapp backed up incase another skiddie defaces us again.

\# We will be able to quickly restore from a backup in seconds ;P

\#-------------------------------------------------------------------------------------

&#x20;

\# Set Vars Here

basedir=/var/www/html

bkpdir=/var/backups

tmpdir=/var/tmp

testmsg=$bkpdir/onuma\_backup\_test.txt

errormsg=$bkpdir/onuma\_backup\_error.txt

tmpfile=$tmpdir/.$(/usr/bin/head -c100 /dev/urandom |sha1sum|cut -d' ' -f1) **\[which is some random hash for a name]{.underline}**

check=$tmpdir/check **\[which is /var/tmp/check]{.underline}**

&#x20;

\# formatting

printbdr()

{

for n in $(seq 72);

do /usr/bin/printf $"-";

done

}

bdr=$(printbdr)

&#x20;

\# Added a test file to let us see when the last backup was run

/usr/bin/printf $"$bdr\nAuto backup backuperer backup last ran at : $(/bin/date)\n$bdr\n" > $testmsg

&#x20;

\# Cleanup from last time.

/bin/rm -rf $tmpdir/.\* $check

&#x20;

\# Backup onuma website dev files.

/usr/bin/sudo -u onuma /bin/tar -zcvf $tmpfile $basedir &

&#x20;

\# Added delay to wait for backup to complete if large files get added.

/bin/sleep 30

&#x20;

\# Test the backup integrity

integrity\_chk()

{

/usr/bin/diff -r $basedir $check$basedir

}

&#x20;

/bin/mkdir $check

/bin/tar -zxvf $tmpfile -C $check

if \[\[ $(integrity\_chk) ]]

then

\# Report errors so the dev can investigate the issue.

/usr/bin/printf $"$bdr\nIntegrity Check Error in backup last ran : $(/bin/date)\n$bdr\n$tmpfile\n" >> $errormsg

integrity\_chk >> $errormsg

exit 2

else

\# Clean up and save archive to the bkpdir.

/bin/mv $tmpfile $bkpdir/onuma-www-dev.bak

/bin/rm -rf $check .\*

exit 0

fi

This variable portion would tell us where the files are stored or something. Take note of these folders.

&#x20;

This portion here just does formatting, making sure that it is in sequence(?)

&#x20;

Do a bunch of printing stuff, but most notably the next part.

&#x20;

&#x20;

&#x20;

Takes all .files and deletes it completely.

&#x20;

Ok this is more interesting, it tars this file. Afterwards, the tar file is left within the /var/tmp/check/

&#x20;

Sleep 30s after. This seems to be a window of which we can act or something, completely no reason to put this here.

&#x20;

Integrity check function to check on differences.

&#x20;

Tars all of this together, within the file. Checks whether or not there are differences regarding this.

If there is a difference, **then the file does not seem to be removed, just highlighted.**

&#x20;

Since onuma is the one that tars the file in the first place, that means we can execute it.

&#x20;

I had a rough sensing of the code here. We need to make some kind of file that would replace the file that is created every 30 seconds or so, and then because if we replace it, the difference would definitely be different.

&#x20;

The file will then be left there.

&#x20;

What's more is that the file is tar'd using onuma, meaning that we would potentially execute this file.

This would mean we need some kind of binary or executable in order to make sure that we are allowed to execute this.

&#x20;

The machine is also 32-bit, as I had to use pspy32.

&#x20;

So if we can replace the file that is created, we are able to simply just get a executable SUID that's just there. This is because again, onuma user is being used to tar the file together.

&#x20;

From here, I looked up a walkthrough as I was really unsure of what to do at this point.

&#x20;

Turns out we need to move and compile it in a different directory. This is because of the fact that the SETUID bit will not be set.

!\[\[53\_TartarSauce\_image0035.png]]

This is the code that would be used.

&#x20;

Now, we need to compile it.

!\[\[53\_TartarSauce\_image0036.png]]

Now Tar it togerther. This includes the entire directory it was included in.

&#x20;

!\[\[53\_TartarSauce\_image0037.png]]

Transfer it over to the machine within the /var/tmp file.

Wait for the file to appear.

It might take a while as the script runs every 5 minutes.

&#x20;

!\[\[53\_TartarSauce\_image0038.png]]

Do this.

And now wait for the /var/check directory to appear.

From there, run it and we should be root (unless you messed up somewhere like me).

!\[\[53\_TartarSauce\_image0039.png]]

&#x20;

!\[\[53\_TartarSauce\_image0040.png]]

This should give you the rootshell.

&#x20;

So turns out, **I forgot to chmod and elevate the privileges of this suid binary.**

&#x20;

This is an important step as it would allow us to basically become root.

&#x20;

I redid the process again.

!\[\[53\_TartarSauce\_image0041.png]]

&#x20;

!\[\[53\_TartarSauce\_image0042.png]]

&#x20;

&#x20;

!\[\[53\_TartarSauce\_image0043.png]]

&#x20;

{width="6.84375in" height="8.635416666666666in"}

Finally!

&#x20;

Grab the user flag.

&#x20;

Learnt a lot in this box, despite having to look up a walkthrough. Was really fun though.
