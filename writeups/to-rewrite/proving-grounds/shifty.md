# Shifty

Shifty

Tuesday, 22 March 2022

3:12 pm

These ports are open:

!\[\[29\_Shifty\_image001.png]]

&#x20;

!\[\[29\_Shifty\_image002.png]]

The web servers are kind of useless, but most interesting to me was the memcache, which is a sort of memory organiser for the website.

&#x20;

When doing a memcdump, it interestingly gives us all of the possible session tokens that have accessed the server.

So there's a small hint here not to use gobuster, as this completely destroys the stuff we have.

&#x20;

This web cache is for port 5000, which only contains one single login page, with the password being admin:admin, and no other possible advancements it seems.

!\[\[29\_Shifty\_image003.png]]

&#x20;

!\[\[29\_Shifty\_image004.png]]

&#x20;

This would seem to add on the more times we visit the web server.

&#x20;

From this, it seems we can inject our own session cookies:

&#x20;

!\[\[29\_Shifty\_image005.png]]

&#x20;

!\[\[29\_Shifty\_image006.png]]

Interesting, wonder if this could be used for some form of RCE.

&#x20;

Googling more led me to believe that this is a pickling exploit that is present within the web application.

!\[\[29\_Shifty\_image007.png]]

&#x20;

Download this exploit, and from here we can get an RCE!

!\[\[29\_Shifty\_image008.png]]

&#x20;

!\[\[29\_Shifty\_image009.png]]

&#x20;

This would work in installing other exploits onto the device, such as other forms of reverse shells.

!\[\[29\_Shifty\_image0010.png]]

&#x20;

!\[\[29\_Shifty\_image0011.png]]

&#x20;

!\[\[29\_Shifty\_image0012.png]]

&#x20;

Within the opt directory, there was this backup.py file, and I'm guessing this is some form of cronjob. It's owned by root and hence can be executed when some condition is met.

&#x20;

I ran a linpeas to determine whether or not this is something that we need to consider.

Could still be another rabbit hole.

!\[\[29\_Shifty\_image0013.png]]

&#x20;

!\[\[29\_Shifty\_image0014.png]]

&#x20;

The video group is not very interesting because of the fact that there are no other users that are currently logged on with us.

!\[\[29\_Shifty\_image0015.png]]

&#x20;

The backup.py file is definitely something that we need to trigger.

!\[\[29\_Shifty\_image0016.png]]

This seems to take the data file within the directory and encrypts the file inside with some kind of stuff.

&#x20;

!\[\[29\_Shifty\_image0017.png]]

Within the data file, there seem to be many different hashes as names.

&#x20;

Within the file, it is encrypted nonsense.

It seems that we can run this script too, and take files to encode.

&#x20;

!\[\[29\_Shifty\_image0018.png]]

&#x20;

Most importantly, I was wondering what kind of file can I take in order to encrypt and take back. It does not allow us to be able to read any root files.

&#x20;

Then I realised, I could hijack the python library perhaps?

This takes a library from des, and perhaps hijack in order to let someone execute it and

!\[\[29\_Shifty\_image0019.png]]

I ran PSPY to check whether or not this was being executed anytime soon.

&#x20;

However, it does not seem so.

&#x20;

Seems like Python Library hijacking is the most likely solution, as I cannot edit the script either.

We cannot edit the des.py either, leaving just the hashlib library to try.

This does not work either, as we cannot edit the file.

!\[\[29\_Shifty\_image0020.png]]

These are the possible files, of which all of them cannot be edited.

&#x20;

!\[\[29\_Shifty\_image0021.png]]

&#x20;

!\[\[29\_Shifty\_image0022.png]]

Wondered if I could perhaps decrypt the stuff that has been written within that.

&#x20;

We clearly need to decrypt these files somehow.

Probably one of them has the password within them or some kind of password.

&#x20;

Openssl can do the trick here. I copied one of the files and got it to my home directory, ran this command and decrypted it!

&#x20;

!\[\[29\_Shifty\_image0023.png]]

&#x20;

!\[\[29\_Shifty\_image0024.png]]

Seems to be the web page... Let's take all of the files and run it through this command.

&#x20;

!\[\[29\_Shifty\_image0025.png]]

The next file I took had the root hash!

&#x20;

Ran a john on it while I continued decrypting the files.

&#x20;

!\[\[29\_Shifty\_image0026.png]]

Another file had the ssh key, and I was confident that the private key would be here somewhere.

&#x20;

Eventually...

!\[\[29\_Shifty\_image0027.png]]

&#x20;

!\[\[29\_Shifty\_image0028.png]]

&#x20;

!\[\[29\_Shifty\_image0029.png]]

&#x20;

&#x20;

Got sidetracked by python libraries and stuff and cronjobs, but still got it anyways.

&#x20;

&#x20;
