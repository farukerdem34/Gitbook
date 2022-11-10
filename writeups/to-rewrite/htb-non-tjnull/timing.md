# Timing

Timing

Thursday, 14 April 2022

1:46 am

A tricky machine that required the use of multiple exploits, of which one was the PHP trick whereby we can use an LFI to read more about how the web application works and from there be able to log in.

&#x20;

From there, we can get an LFI to trigger a rev shell and then SSH in as the user through dumping some keys into his ssh file.

&#x20;

Once that is done, we are able to find a binary of which we can use to create files. Since we can create files, we can create a symlink for a file to change other files through that.

&#x20;

This would allow us to create a symlink from the user directory to that of the keys included within the home directory of the user.

&#x20;

This would let us change the authorized\_keys file of the root user and be able to append ours into it, allowing us root access.

&#x20;

Interesting PE vector.
