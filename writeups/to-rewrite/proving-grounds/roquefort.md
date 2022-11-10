# Roquefort

Roquefort

Friday, 1 April 2022

11:20 am

Use Gitea 1.7.5 RCE to gain foothold.

Afterwards, simply look for the cron job that uses certain commands, and find out that we can actually write into a path that cron uses to gain root shell.
