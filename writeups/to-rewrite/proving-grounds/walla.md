# Walla

Walla

Thursday, 17 March 2022

11:13 am

Linux machine that has telnet enabled for some reason...but had default credentials of admin:secret.

&#x20;

From there, there's a console that can we can use to netcat a shell.

!\[\[15\_Walla\_image001.png]]

&#x20;

Afterwards, linpeas determines we have write privileges over some services, leading to some exploitation of echoing in some /bin/sh into the service, then executing it again.

&#x20;

Either that, or we can replace the one python script we can sudo with another one.

!\[\[15\_Walla\_image002.png]]

&#x20;
