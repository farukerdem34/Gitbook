# Fail

Fail

Friday, 18 March 2022

4:13 pm

Rsync box.

&#x20;

Upload some SSH keys, can be your own if desperate and then SSH into machine as fox. Follow commands on hacktricks for rsync enumeration.

&#x20;

!\[\[20\_Fail\_image001.png]]

&#x20;

Discover that we're part of the fail2ban group, and we can edit the actions that are taken when the server bans us from the website. We can make /etc/passwd executable and hence allow for us to append in our own user as a root user.

From there, same as per xposedAPI. Fail2ban is a new exploit I've never used before.

!\[\[20\_Fail\_image002.png]]

&#x20;
