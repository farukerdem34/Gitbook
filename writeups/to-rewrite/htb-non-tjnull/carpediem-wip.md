# CarpeDiem (WIP)

CarpeDiem (WIP)

Tuesday, June 28, 2022

10:16 AM

Nmap scan:

!\[\[33\_CarpeDiem (WIP)\_image001.png]]

&#x20;

!\[\[33\_CarpeDiem (WIP)\_image002.png]]

&#x20;

Port 80:

!\[\[33\_CarpeDiem (WIP)\_image003.png]]

&#x20;

We can add that to our /etc/hosts file and try to find more directories that are within this website.

&#x20;

A directory enumeration returns nothing, but a vhost enumeration returns one other potential website we can head to:

!\[\[33\_CarpeDiem (WIP)\_image004.png]]

&#x20;

!\[\[33\_CarpeDiem (WIP)\_image005.png]]

&#x20;

This brings us to some kind of motorcycle store.

!\[\[33\_CarpeDiem (WIP)\_image006.png]]

There's a p parameter, and LFI and RFI do not work here.

The pages use a hash to distinguish which product they are referring to, which is a bit odd.

!\[\[33\_CarpeDiem (WIP)\_image007.png]]

&#x20;

There are logins and admin panels present, but we have no known credentials.

&#x20;

When updating the profiles of a user we created, we can see the parameters here.

!\[\[33\_CarpeDiem (WIP)\_image008.png]]

&#x20;

There appears to be a login\_type, of which I have changed the value to 1 for testing.

Afterwards, we can access the admin panel.

&#x20;

!\[\[33\_CarpeDiem (WIP)\_image009.png]]

&#x20;

Within the settings, it appears there is quarterly sales report .xlsx file.

!\[\[33\_CarpeDiem (WIP)\_image0010.png]]

&#x20;

So the vulnerability is the file upload.

We can try to upload a file to this.

&#x20;

When we view the request, we can see that we need to include the form data.

So this could be the way.

&#x20;

!\[\[33\_CarpeDiem (WIP)\_image0011.png]]

&#x20;

&#x20;

!\[\[33\_CarpeDiem (WIP)\_image0012.png]]

&#x20;

!\[\[33\_CarpeDiem (WIP)\_image0013.png]]

&#x20;

Shell:

!\[\[33\_CarpeDiem (WIP)\_image0014.png]]

&#x20;

Upload a PHP reverse shell.

Then curl the URL and we should get one.

&#x20;

{width="9.75in" height="2.3541666666666665in"}

&#x20;

!\[\[33\_CarpeDiem (WIP)\_image0016.png]]

&#x20;

We are in a container, it seems.

!\[\[33\_CarpeDiem (WIP)\_image0017.png]]

&#x20;

We can find some root passwords and stuff.

&#x20;

There also is this.

!\[\[33\_CarpeDiem (WIP)\_image0018.png]]

&#x20;

Perhaps we need to chisel some things over, to get the MySQL database accessible to us.

&#x20;

Also, there are other things like this.

!\[\[33\_CarpeDiem (WIP)\_image0019.png]]

&#x20;

So it seems that this server has a port 3306 listening, and we should be using chisel to try and access this MySQL database.

So we know that 171.17.0.3 has the SQL database and 172.17.0.5 is something completely different.

&#x20;

There seem to be a load of containers within this machine, and we need to find out which is which. First thing we can do it is an nmap scan of every single domain that is present on this machine to see which is alive.

&#x20;

We can download an nmap binary to the amchine and then do this command:

!\[\[33\_CarpeDiem (WIP)\_image0020.png]]

&#x20;

Results:

!\[\[33\_CarpeDiem (WIP)\_image0021.png]]

&#x20;

!\[\[33\_CarpeDiem (WIP)\_image0022.png]]

&#x20;

Seems like there are a load of other things on this machine.

&#x20;

We can chisel and proxychains the rest.

!\[\[33\_CarpeDiem (WIP)\_image0023.png]]

&#x20;

!\[\[33\_CarpeDiem (WIP)\_image0024.png]]

&#x20;

!\[\[33\_CarpeDiem (WIP)\_image0025.png]]

&#x20;

Afterwards, we can access the different ports that are present.

Then we can configure foxyproxy to use proxychains to view the stuff on these websites.

&#x20;

MySQL:

{width="8.510416666666666in" height="3.4791666666666665in"}

&#x20;

Since we were root on this MySQL user, we can theoretically abuse the UDF functions of this. Nothing of use within this one.

&#x20;

Port 443 on 172.17.0.2:

!\[\[33\_CarpeDiem (WIP)\_image0027.png]]

&#x20;

This is yet another domain.

There are logins and stuff but we can't do much without some form of credentials.

&#x20;

Additionally, there is a trudesk.carpediem.htb on 172.17.0.6 on port 8118.

&#x20;

This box has a lot of leads.

Let's look into that MongoDB database first.

&#x20;

I tried to access stuff like backdrop and truman, and it worked.

{width="3.9479166666666665in" height="0.65625in"}

&#x20;

!\[\[33\_CarpeDiem (WIP)\_image0029.png]]

&#x20;

!\[\[33\_CarpeDiem (WIP)\_image0030.png]]

&#x20;

Lots of information here.

{width="9.854166666666666in" height="2.25in"}

&#x20;

&#x20;

We can actually modify the password and stuff from MongoDB as well.

!\[\[33\_CarpeDiem (WIP)\_image0032.png]]

&#x20;

This would modify the admin password of trudesk and we would be able to login to that website.

Not sure what else there is to do here though, what we should be doing is looking at the zoiper thing.

&#x20;

There are some initial credentials too, so what's the default credential?

&#x20;

When logging into the trudesk with our alterered password, we can see the tickets.

!\[\[33\_CarpeDiem (WIP)\_image0033.png]]

&#x20;

!\[\[33\_CarpeDiem (WIP)\_image0034.png]]

There seems to be mention of this Horace Flaccus, which would become the username of hflaccus. This user apparently has a default credential used.

Basically, Zoiper this and find the voicemail, that's our next step.

&#x20;

&#x20;

&#x20;
