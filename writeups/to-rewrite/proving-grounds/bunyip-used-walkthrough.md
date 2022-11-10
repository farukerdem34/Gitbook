# BunyIP (Used walkthrough)

BunyIP (Used walkthrough)

Thursday, June 16, 2022

9:30 PM

Nmap scan:

!\[\[57\_BunyIP (Used walkthrough)\_image001.png]]

&#x20;

Port 80:

!\[\[57\_BunyIP (Used walkthrough)\_image002.png]]

&#x20;

Looks like any old regular website.

&#x20;

&#x20;

Port 8080:

!\[\[57\_BunyIP (Used walkthrough)\_image003.png]]

&#x20;

Interesting, this is a remote code executor that would seem to run JS code based on what kind of API-KEY we give it.

So the hash basically needs to be correct for the API Key that we want to execute.

&#x20;

When checking Burpsuite, the code is base64 encoded while the sig is just the hash sent.

!\[\[57\_BunyIP (Used walkthrough)\_image004.png]]

&#x20;

The attack that needs to be used on this is called a hash extender attack, whereby there is the hash and one of the parts of the original plaintext that goes into the hash, in this case being the code that is executed.

&#x20;

As such, we can calculate the hash that is correct without knowing the API Key, all we need is the hash and the code we want to execute.

&#x20;

Basically, we can generate hashes with the secret still there and allow it to be valid when trying to execute code on this machine.

&#x20;

Let's test it out with the basic code they gave us which prints hello world.

[https://github.com/iagox86/hash\_extender](https://github.com/iagox86/hash\_extender)

&#x20;

Right, so we already have a valid signature present here.

!\[\[57\_BunyIP (Used walkthrough)\_image005.png]]

&#x20;

The next step would be to try to recreate this signature with our own programs without ever knowing the API Key used here.

So we know that the hash of aaa8111... is hash = md5(api\_key||code).

&#x20;

Let's first find out how it's hashing the data.

!\[\[57\_BunyIP (Used walkthrough)\_image006.png]]

&#x20;

!\[\[57\_BunyIP (Used walkthrough)\_image007.png]]

&#x20;

So the portion that is being appended would be the SECRET in cleartext and then the '|' character.

&#x20;

And we know that the text is something like \<API-KEY>|code.

&#x20;

Now, we can attempt to extend this hash with whatever appending that we want to do.

So for this attack, we would need to append to the message we are sending. In this case, it would mean the attack vector is simple, we need to append code to the existing code and be able to generate a good hash.

&#x20;

For now, we can test it out with something simple like hello('test');

&#x20;

We don't need to build it from scratch, just need to add on to an existing python repository that uses this attack:

[https://github.com/cbornstein/python-length-extension](https://github.com/cbornstein/python-length-extension)

&#x20;

Now, reading the code, it appears that this has two scripts

* Len\_ext.py
* Pymd5.py

&#x20;

We can combine it into one script by just copying and pasting the entire pymd5 portion into our edited exploit.

So next, we would need to set up a method of which we can send requests to the website, in which case it is a JSON object with the code encoded in base64 and the hash key that is used.

&#x20;

!\[\[57\_BunyIP (Used walkthrough)\_image008.png]]

&#x20;

This bit shall do.

Now, we need to define all the variables that we need

* Current hash
* Current message
* Message we want to append
* Length of the key that is being used (in this case, it's 37 characters)

&#x20;

{width="6.625in" height="2.3229166666666665in"}

&#x20;

Then, we need to do the functions to make the new hash.

!\[\[57\_BunyIP (Used walkthrough)\_image0010.png]]

&#x20;

A little explanation for this:

* First we take our appended JS code and then just keep it there
* Afterwards, adapting from the function from the repository, we would then need to make our new code. Of which case, we can take the current code, and append the padding of the length of the existing key | message
  * !\[\[57\_BunyIP (Used walkthrough)\_image0011.png]]

The above picture would be the value of our extended\_code, of which it would be one that used to exploit the hash.

&#x20;

* Next, we would just take our appended code and add it to the end there as we can see
* Afterwards, we would need to update the hash with our appended code
* Then, we would create another hash using the extended\_hash and updating it, creating the new hash that we pass on.

&#x20;

The result is this:

{width="4.0625in" height="0.90625in"}

&#x20;

So we have successfully extended the hash and exploited this vulnerability. Now, we need to create a new append message to gain a reverse shell.

Head over to revshells.com to generate one in Node JS for port 443.

!\[\[57\_BunyIP (Used walkthrough)\_image0013.png]]

&#x20;

Shell:

!\[\[57\_BunyIP (Used walkthrough)\_image0014.png]]

&#x20;

{width="9.75in" height="1.78125in"}

&#x20;

&#x20;

Flag:

!\[\[57\_BunyIP (Used walkthrough)\_image0016.png]]

&#x20;

PE:

First thing we take note of is that unknown group, which is called safe-backup.

!\[\[57\_BunyIP (Used walkthrough)\_image0017.png]]

&#x20;

We can see this, so perhaps we can create a whole backup of the /root directory.

&#x20;

Backup:

!\[\[57\_BunyIP (Used walkthrough)\_image0018.png]]

&#x20;

Then we can decrypt this:

{width="8.760416666666666in" height="2.4791666666666665in"}

&#x20;

Now we can basically generate another /root file, but we still cannot access it.

So let's think about this, it creates a password-encrypted thing whereby we can exploit it.

Also, this tool is possible to make backups and import them into the system.

&#x20;

This gives me the idea to just replace the SSH keys within the /root directory.

Let's generate some keys.

{width="8.447916666666666in" height="4.34375in"}

&#x20;

{width="3.8125in" height="1.4375in"}

&#x20;

We can then save this backup.

!\[\[57\_BunyIP (Used walkthrough)\_image0022.png]]

&#x20;

We need to rename it to fit root's directory.

{width="5.375in" height="0.59375in"}

&#x20;

Afterwards, once transferred using wget, we just need to import the key.safe as well, and then create a symbolic link of which would put the key within the SSH Directory of root.

{width="9.604166666666666in" height="2.53125in"}

&#x20;

Then we can SSH in.

!\[\[57\_BunyIP (Used walkthrough)\_image0025.png]]

&#x20;

Flag:

!\[\[57\_BunyIP (Used walkthrough)\_image0026.png]]

&#x20;

This box was hard, and even the PE was considered difficult. Thank god they give walkthroughs else I would be unable to solve this one as I couldn't get the foothold.

&#x20;

The things I've learnt from this are:

* Always check for unique hash attacks, and suspicious information that is given
* See how the safe-backup does things, and how that can be exploited through certain means.
* Try Harder.

&#x20;

&#x20;

&#x20;
