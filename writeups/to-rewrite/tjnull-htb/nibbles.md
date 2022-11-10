# Nibbles

Nibbles

Tuesday, 1 February 2022

1:17 am

**\[Nibbles (Web based)]{.underline}**

This has a nibblesblog directory.

No further directories present.

There is a feed.php file present. Everything is pointing towards 10.10.10.134...&#x20;

Upon brute forcing the login, it seems that we became blacklisted. Interestingly, I found these:

Seems that we are allowed to upload some form of templates into the website.&#x20;

We get blacklisted if we try anything funny... hydra does not work it seems.

Nibbleblog 4.0.3 was found when I tried to access the README file. This version has a web shell vulnerability.

Guess the password of nibbles to prevent being blocked somehow.&#x20;

Rest of the steps:

* Upload php shell
* Find the sudo and what we can execute, which would be a script within a certain directory
* Append the script with "/bin/bash -i" >> monitor.sh
* Run it and we are root!
