# Hutch (PG)

Hutch (PG)

Thursday, 17 March 2022

3:27 pm

Use Ldapsearch to get a valid user and password embedded within the lines. Search for word 'password' and go through them one by one.

&#x20;

From there, nikto scans or MSF scans reveal that this server has webdav enabled. This would mean we can use cadaver to put .asp shells on there to gain access into the machine easily to get user flag.

&#x20;

!\[\[11\_Hutch (PG)\_image001.png]]

&#x20;

Run a winpeas, and we can find out the administrator password there.

There's LAP installed, and we can run ldap search to dump it.

!\[\[11\_Hutch (PG)\_image002.png]]

&#x20;

!\[\[11\_Hutch (PG)\_image003.png]]

From there, we can evil-winrm in.

&#x20;

&#x20;
