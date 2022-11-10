# Craft

Craft

Friday, 1 April 2022

11:21 am

Simple Windows machine that involves using macros to first inject a powershell reverse shell.

&#x20;

Afterwards, lateral escalation through become the user apache through a cmd.php on the web server itself.

&#x20;

User apache has SeImpersonatePrivileges on, hence we can use PrintSPoofer for easy admin access.

&#x20;

&#x20;
