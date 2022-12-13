# Seventeen (WIP)

Seventeen (WIP)

Thursday, June 23, 2022

1:51 AM

Nmap scan:

!\[\[29\_Seventeen (WIP)\_image001.png]]

&#x20;

!\[\[29\_Seventeen (WIP)\_image002.png]]

&#x20;

Port 80:

!\[\[29\_Seventeen (WIP)\_image003.png]]

&#x20;

We can add the domain to our hosts file.

!\[\[29\_Seventeen (WIP)\_image004.png]]

&#x20;

Gobust:

!\[\[29\_Seventeen (WIP)\_image005.png]]

&#x20;

!\[\[29\_Seventeen (WIP)\_image006.png]]

&#x20;

Nothing much here.

We can try to fuzz subdomains.

&#x20;

Subdomain fuzzing:

!\[\[29\_Seventeen (WIP)\_image007.png]]

&#x20;

!\[\[29\_Seventeen (WIP)\_image008.png]]

&#x20;

New domain found, let's add to our hosts file.

!\[\[29\_Seventeen (WIP)\_image009.png]]

&#x20;

This is some examination management system, which already looks vulnerable.

!\[\[29\_Seventeen (WIP)\_image0010.png]]

&#x20;

We can try the python script.

Editing exploit:

!\[\[29\_Seventeen (WIP)\_image0011.png]]

&#x20;

Doesn't work.

&#x20;

Let's directory gobust this website again.

&#x20;

Gobuster:

!\[\[29\_Seventeen (WIP)\_image0012.png]]

&#x20;

!\[\[29\_Seventeen (WIP)\_image0013.png]]

&#x20;

The admin panel is disabled...

We can try to access these directories.

&#x20;

What's interesting is that...we have some form of local services being hosted.

!\[\[29\_Seventeen (WIP)\_image0014.png]]

&#x20;

It seems to be directed at port 8081 on the localhost...which is interesting to say the least.

&#x20;

When trying different exploits, we can see this one works..?

!\[\[29\_Seventeen (WIP)\_image0015.png]]

&#x20;

[https://raw.githubusercontent.com/twseptian/rce-authenticated-from-exploit-db/main/rce-auth.py](https://raw.githubusercontent.com/twseptian/rce-authenticated-from-exploit-db/main/rce-auth.py)

&#x20;

Interesting.

Doesn't really work but whatever.

We can see how it sort of works and deduce the CMS that is being used.

[https://www.exploit-db.com/exploits/50725](https://www.exploit-db.com/exploits/50725)

&#x20;

Then we can find this exploit and exploit it via SQLMap.

!\[\[29\_Seventeen (WIP)\_image0016.png]]

&#x20;

!\[\[29\_Seventeen (WIP)\_image0017.png]]

&#x20;

Dumping database:

{width="3.6979166666666665in" height="0.5729166666666666in"}

&#x20;

{width="7.052083333333333in" height="2.1666666666666665in"}

&#x20;

{width="6.8125in" height="1.125in"}

&#x20;

!\[\[29\_Seventeen (WIP)\_image0021.png]]

&#x20;

Oldmanagement? We can gobust that for now.

&#x20;

This table didn't really have anything, so I proceeded to just dump the entire database instead.

&#x20;

&#x20;

&#x20;

&#x20;

OldManagement:

!\[\[29\_Seventeen (WIP)\_image0022.png]]

&#x20;

We find a login page!

&#x20;

Gobust of oldmanagement:

{width="9.791666666666666in" height="0.8125in"}

&#x20;

!\[\[29\_Seventeen (WIP)\_image0024.png]]

&#x20;

There's a /db file which is always interesting.

&#x20;

Continue SQLMap:

{width="3.3541666666666665in" height="0.53125in"}

&#x20;

{width="4.90625in" height="1.09375in"}

&#x20;

We can dump db\_sfms:

{width="5.083333333333333in" height="0.7916666666666666in"}

&#x20;

We get a student number:

{width="8.260416666666666in" height="1.46875in"}

&#x20;

Credentials:

{width="9.166666666666666in" height="1.96875in"}

&#x20;

Only one hash cracks:

!\[\[29\_Seventeen (WIP)\_image0030.png]]

&#x20;

For stuid 31234.

&#x20;

Login:

!\[\[29\_Seventeen (WIP)\_image0031.png]]

&#x20;

!\[\[29\_Seventeen (WIP)\_image0032.png]]

&#x20;

PDF:

!\[\[29\_Seventeen (WIP)\_image0033.png]]

&#x20;

Interesting, there's a file management application running on this system, and we need to try and upload a shell it seems.

&#x20;

Upload exploit:

!\[\[29\_Seventeen (WIP)\_image0034.png]]

&#x20;

We can upload PHP files, now we just need to execute it somehow.

&#x20;

Remember the /files directory? That must be the file system the PDF is talking about.

!\[\[29\_Seventeen (WIP)\_image0035.png]]

&#x20;

However, it seems that executing PHP code it not very possible within this directory. We need to find a way to execute PHP code within the browser to gain a shell. Perhaps the mailmaster thing could execute code.

&#x20;

WebMail:

!\[\[29\_Seventeen (WIP)\_image0036.png]]

&#x20;

When searching for roundcube, there are some exploits:

{width="7.8125in" height="3.4166666666666665in"}

&#x20;

It is unlikely that this uses the RCE exploits, because the page source does state this should be at least version 1.

Page source:

!\[\[29\_Seventeen (WIP)\_image0038.png]]

&#x20;

SQLMap confirms this:

{width="3.78125in" height="1.40625in"}

&#x20;

Gobuster:

{width="9.822916666666666in" height="0.7916666666666666in"}

&#x20;

!\[\[29\_Seventeen (WIP)\_image0041.png]]

&#x20;

Quite a lot of stuff is given to us.

&#x20;

/installer:

We can find this interesting web directory available to us:

!\[\[29\_Seventeen (WIP)\_image0042.png]]

&#x20;

Looks like we can create configs.

&#x20;

We can find the database password:

!\[\[29\_Seventeen (WIP)\_image0043.png]]

&#x20;

When looking at the possible CVEs for this, I came across this one:

[https://github.com/DrunkenShells/Disclosures/tree/master/CVE-2020-12640-PHP%20Local%20File%20Inclusion-Roundcube](https://github.com/DrunkenShells/Disclosures/tree/master/CVE-2020-12640-PHP%20Local%20File%20Inclusion-Roundcube)

&#x20;

This exploit would abuse the file inclusion plugin that can be enabled through the /installer page, of which we can execute PHP code.

This gives me an idea, to be able to upload the php shell through the School File System and then execute it using the wondermail exploit.

So let's try it out.

&#x20;

Firstly, upload the shell we want to execute and set up a listener port. Make it the same name as the student ID

&#x20;

&#x20;

Now, based on the PoC, the mastermailer directory should be in /var/www/html/mastermailer, hence this school thingy should be in /var/www/html/oldmanagement/files/31234/31234.php.

&#x20;

So now we need to generate the request to update the config that includes the plugin Name parameters.

How to do this is basically go to the plugins directory and then just include any of the plugins and hit save config to generate it.

&#x20;

It would look something like this.

!\[\[29\_Seventeen (WIP)\_image0044.png]]

&#x20;

Then we just need to alter the name of this plugin to the PHP file that we want to execute and send this POST request.

Afterwards, just try to access the main login page to gain a shell.

&#x20;

Right, so this didn't really work out very well in getting us the RCE. This was because of the fact that well, the box was a weird boot up.

After resetting and trying again, the box seems to function properly now.

{width="9.822916666666666in" height="2.5729166666666665in"}

&#x20;

Within this machine, we can see that we are in a container.

!\[\[29\_Seventeen (WIP)\_image0046.png]]

&#x20;

Looking at /var/www/html:

There seem to be a lot of directories within this machine.

!\[\[29\_Seventeen (WIP)\_image0047.png]]

&#x20;

Within one of the files, we can actually find credentials:

!\[\[29\_Seventeen (WIP)\_image0048.png]]

&#x20;

Interesting!

Now we have a set of credentials for a database, and it seems it is for the root user within this container.

We don't have much use for this container, so we can try to brute force ssh with some basic names and this password.

Eventually, we find that it belongs to the user 'mark'.

&#x20;

We can ssh in and grab the flag.

!\[\[29\_Seventeen (WIP)\_image0049.png]]

&#x20;

PE:

Within this, we can see how there are a load of ports that are present.

!\[\[29\_Seventeen (WIP)\_image0050.png]]

&#x20;

We also find there is another user called kavi within this machine.

{width="4.28125in" height="0.9791666666666666in"}

&#x20;

Within /opt, there are some files located.

!\[\[29\_Seventeen (WIP)\_image0052.png]]

&#x20;

{width="5.59375in" height="4.614583333333333in"}

&#x20;

This script seems to check for the db-logger module being installed.

However, there is limited things we can do with Mark as the user here, and I would like to check if we can access Kavi somehow.

&#x20;

!\[\[29\_Seventeen (WIP)\_image0054.png]]

&#x20;

So this takes a package located in this directory and then proceeds to download it. We can use this to execute some form of node.js code.

When we run the script, it seems that it actually installs the packages needed.

{width="5.34375in" height="1.8020833333333333in"}

&#x20;

!\[\[29\_Seventeen (WIP)\_image0056.png]]

&#x20;

Then we have this package thing, of which we can actually modify somehow.

First, let's grab a reverse shell from revshells.com.

!\[\[29\_Seventeen (WIP)\_image0057.png]]

&#x20;

Then, let's try to replace this package with our own js code and execute the shell accordingly.

!\[\[29\_Seventeen (WIP)\_image0058.png]]

&#x20;

Afterwards, we can replace the loglevel.js within the machine with our own.

!\[\[29\_Seventeen (WIP)\_image0059.png]]

&#x20;

!\[\[29\_Seventeen (WIP)\_image0060.png]]

&#x20;

Then we just need to package this again.

!\[\[29\_Seventeen (WIP)\_image0061.png]]

&#x20;

Afterwards, simply copy this entire directory back into the .npm file and sudo execute the bash script again.

We may get this.

!\[\[29\_Seventeen (WIP)\_image0062.png]]

&#x20;

This is because we basically changed the entire thing to a reverse shell, so at least we know it works.

&#x20;

This indicates to me that the js shell we had is not working properly.

WIP.

&#x20;

&#x20;
