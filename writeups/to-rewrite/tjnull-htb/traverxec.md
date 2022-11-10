# Traverxec

Traverxec

Saturday, 12 February 2022

9:51 pm

&#x20;

> I did this machine at work, so no fancy screenshots or whatever.
>
> &#x20;
>
> To outline the steps:

1. Enumerate with Nmap, discover port SSH and port HTTP.
2. Identify that HTTP is running a nostromo web server.
3. Directory enumerate, and find out there's a Linux MD5 password hash within the files somewhere.
4. Decrypt the password.
5. Check the httpconf, and determine that the home directory is \~david.
6. Get the zip file, and using the password unzip it.
7. It should contain id\_ras keys.
8. SSH2John
9. Use the decrypted RSA key to SSH in as the user, get the user flag.
10. Check the bin file within user directory.
11. There is a file that is executable by the user.
12. JournalCTl is the SUID that can be used.
13. Edit the terminal editors to include /bin/bash within the script and execute it.
14. You are now root!
