# Hawat

Hawat

Wednesday, 23 March 2022

6:13 pm

This machine has easy credentials to log into a hidden cloud directory to find the source code of one of the applications.

&#x20;

From there, we can look around and see that within the main file, there is SQL queries bieng passed into the web server on the back-end.

&#x20;

!\[\[37\_Hawat\_image001.png]]

This is a vector for SQL injection and we can draw out databases with this or gain a command shell.

&#x20;

The SQL query is not sanitised and can be abused for this purpose.

&#x20;

!\[\[37\_Hawat\_image002.png]]

This is due to this function existing, whereby we can pass in our own stuff.

!\[\[37\_Hawat\_image003.png]]

This causes the website to sleep for 5 seconds.

&#x20;

Now, we can use this to perhaps write some files within the directories.

!\[\[37\_Hawat\_image004.png]]

&#x20;

Essentially, this is a URL encoded version of the cmd.php.

UNION SELECT "\<php shell here>" INTO OUTFILE '/srv/http/cmd.php'

&#x20;

From here, the shell would be uploaded onto the other port and we can get RCE.

!\[\[37\_Hawat\_image005.png]]

&#x20;

From here, download a PHP shell and reverse shell in on port 443.

!\[\[37\_Hawat\_image006.png]]

&#x20;

&#x20;
