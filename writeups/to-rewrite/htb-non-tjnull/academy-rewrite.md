# Academy

Academy

Friday, 15 April 2022

8:20 pm

1. Change certain roleid parameters to access an administrator page.
2. Access hidden web page and discover that it is vulnerable to a certain laravel RCE because it exposes the API token
3. From there, we can gain a shell.
4. Password was hidden in a folder picked up by linpeas. This would let us capture the user flag
5. From there, the user we change to has access to the adm folder. Another user on the machine was part of the sudo group.
6. As the adm user, we can use aureport to read out the passwords and stuff from the /var/logs folders. We can then su to the sudo user.
7. The sudo user can sudo one binary found on GTFOBins to become root and capture that flag.
