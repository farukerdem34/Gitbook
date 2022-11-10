# Mantis

Mantis

Sunday, June 26, 2022

8:24 PM

Nmap scan:

!\[\[63\_Mantis \_image001.png]]

&#x20;

!\[\[63\_Mantis \_image002.png]]

&#x20;

We can gobuster port 80:

!\[\[63\_Mantis \_image003.png]]

&#x20;

!\[\[63\_Mantis \_image004.png]]

&#x20;

And find bugtracker.

!\[\[63\_Mantis \_image005.png]]

&#x20;

This is mantis bugtracker, and it has public exploits for it.

&#x20;

I tested a user account:

!\[\[63\_Mantis \_image006.png]]

&#x20;

&#x20;

Nothing really works here regarding accessing the web page.

Interestingly, we can access the admin panel without logging in.

&#x20;

!\[\[63\_Mantis \_image007.png]]

&#x20;

We should be checking on possible ways to exploit this mantis bugtracker, abusing the open admin panel.

WIP.

&#x20;

There should be some kind of exploit regarding this one that would let us view the files and interact with the SQL port that's there and then find some sort of shell.

&#x20;

Right, so we can come across CVE-2017-12419, which would let us basically use a rogue MySQ server to read files.

We can use this PHP code here to run a SQL Server.

[https://mantisbt.org/bugs/view.php?id=23173](https://mantisbt.org/bugs/view.php?id=23173)

[https://github.com/allyshka/Rogue-MySql-Server/blob/master/roguemysql.php](https://github.com/allyshka/Rogue-MySql-Server/blob/master/roguemysql.php)

&#x20;

Then we can visit this URL and see that we are able to read files on this machine.

!\[\[63\_Mantis \_image008.png]]

&#x20;

{width="9.59375in" height="2.625in"}

&#x20;

!\[\[63\_Mantis \_image0010.png]]

&#x20;

Interesting. Now from here, we should be looking for some kind of configuration files or something.

We can view the github repository for this.

[https://github.com/mantisbt/mantisbt](https://github.com/mantisbt/mantisbt)

!\[\[63\_Mantis \_image0011.png]]

&#x20;

Perhaps there is this file here within the machine's own directory.

{width="9.697916666666666in" height="3.4375in"}

&#x20;

From here, we can get some of MySQL password.

{width="8.625in" height="4.333333333333333in"}

&#x20;

Then we can read the bugtracker database.

{width="9.760416666666666in" height="3.6979166666666665in"}

&#x20;

Cracked hash:

!\[\[63\_Mantis \_image0015.png]]

&#x20;

&#x20;

With these credentials, we can look for RCE exploits.

WE can grab the one from exploitdb.

!\[\[63\_Mantis \_image0016.png]]

&#x20;

However, this exploit would also reset the password of the administrator which uses another exploit, of which the current web application is not vulnerable to. In this case, we can still exploit the thing manually using the code as a guideline.

&#x20;

In this case we can visit the manage configuration page.

!\[\[63\_Mantis \_image0017.png]]

&#x20;

Then we need to create two configurations, one to set this variable to 1

!\[\[63\_Mantis \_image0018.png]]

&#x20;

And the next would be to include the reverse shell.

&#x20;

!\[\[63\_Mantis \_image0019.png]]

&#x20;

Then we just need to visit the /workflow\_graph\_img.php page.

&#x20;

{width="9.072916666666666in" height="2.3541666666666665in"}

&#x20;

Flag:

!\[\[63\_Mantis \_image0021.png]]

7557d4098dc2cd16b53443a04365ab39

&#x20;

PE:

Within the user directory, there's this backup.sh and a SQL file.

!\[\[63\_Mantis \_image0022.png]]

&#x20;

We aren't allowed to view this file at all unless we are mantis.

Transferring back to my machine via some base64 encoding, we can view the file.

&#x20;

LinPEAS:

{width="7.25in" height="1.71875in"}

&#x20;

Since this weird sql file has been edited, we can just use pspy64 to view the backup.sh file in action.

!\[\[63\_Mantis \_image0024.png]]

&#x20;

We can see there's a password here.

With this, we can su as mantis.

!\[\[63\_Mantis \_image0025.png]]

&#x20;

{width="9.666666666666666in" height="1.7395833333333333in"}

&#x20;

Aight.

&#x20;

Root shell and flag:

!\[\[63\_Mantis \_image0027.png]]

6b33cc46cd9fa2209ab0124a367a8075

&#x20;

Rooted.

&#x20;
