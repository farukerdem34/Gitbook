# Readys

Readys

Thursday, 17 March 2022

3:08 pm

Tricky box. Done with a walkthrough after hours of failing the priv esc.

&#x20;

Determine the Website's LFI, and dump the Redis credentials.

&#x20;

From there, login to redis and begin putting shells for reverse shelling.

!\[\[17\_Readys\_image001.png]]

&#x20;

From here, abuse a cronjob and then get the shell through making /bin/bash executable for all. Alternatively, create a reverse shell back to our machine. Both should work.

&#x20;

The cronjob works by checking the files within it, and it checks for 3 files each time, and then tar it.

From there we can just create 2 other SUID files and one exploit script that would allow us to be able to execute /bin/bash.

&#x20;

We do chmod +s /bin/bash to allow all to basically change to root, alternatively we can make ourselves root or get a reverse shell again.
