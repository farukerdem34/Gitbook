# Ophiuchi

Ophiuchi

Tuesday, 22 February 2022

1:42 pm

This is a Linux machine.

My IP address is 10.10.16.9.

The target IP address is 10.10.10.227.

&#x20;

Nmap scan this thing as usual.

!\[\[52\_Ophiuchi\_image001.png]]

Tomcat!

&#x20;

Anyways, let's go view this web page.

&#x20;

This is an online YAML parser.

!\[\[52\_Ophiuchi\_image002.png]]

A little empty, so let's run a directory scan while this is going.

&#x20;

Interestingly, inputting a simple 'hello' displays this.

!\[\[52\_Ophiuchi\_image003.png]]

The directory scan seems to be outputting a lot of useless things.

!\[\[52\_Ophiuchi\_image004.png]]

There's a manager directory, which I tried the default credentials of tomcat:s3cret but it did not work out well.

&#x20;

!\[\[52\_Ophiuchi\_image005.png]]

&#x20;

So let's take a closer look at this YAML parser.

&#x20;

YAML stands for Yet Another Markup Language, a serialisation language that is used for configuration files and stuff. Let's try getting some random YAML input inside to parse.

I googled around for some YAML reverse shells that can perhaps be used in this website.

&#x20;

Anyways, I googled and found [this](https://github.com/artsploit/yaml-payload). This seemed to be some kind of payload that triggers a HTTP lookup from this.

&#x20;

I tested it and opened up a HTTP server on my machine.

!\[\[52\_Ophiuchi\_image006.png]]

&#x20;

!\[\[52\_Ophiuchi\_image007.png]]

This worked! Now I just need to find a suitable payload.

&#x20;

Turns out we need to use this .java file to trigger the lookup.

!\[\[52\_Ophiuchi\_image008.png]]

So we can see that this basically execs code, and we can clearly weaponize this easily by including some kind of bash reverse shell.

So this works to get a payload onto the web machine.

I created a simple .sh file using a bash reverse shell to be executed.

&#x20;

!\[\[52\_Ophiuchi\_image009.png]]

&#x20;

Afterwards, I got the git repository on my computer and followed the instructions regarding it.

&#x20;

!\[\[52\_Ophiuchi\_image0010.png]]

&#x20;

Opened up two different HTTP servers, one for the .jar on port 8000 and one for my reverse script on port 80.

Inputted the command to get the .jar file from me.

!\[\[52\_Ophiuchi\_image0011.png]]

&#x20;

!\[\[52\_Ophiuchi\_image0012.png]]

Got a shell on the listener port!

&#x20;

!\[\[52\_Ophiuchi\_image0013.png]]

Stabilised the shell using a TTY, and are successful in initial exploitation.

&#x20;

Went into opt, and found the tomcat directory.

&#x20;

!\[\[52\_Ophiuchi\_image0014.png]]

&#x20;

There is a conf file which normally stores passwords.

!\[\[52\_Ophiuchi\_image0015.png]]

&#x20;

!\[\[52\_Ophiuchi\_image0016.png]]

Let's try an SU.

&#x20;

!\[\[52\_Ophiuchi\_image0017.png]]

Cool.

&#x20;

From here, since we have a password, first thing to check is sudo!

&#x20;

Seems we can run sudo on this file.

!\[\[52\_Ophiuchi\_image0018.png]]

This would be using Go...

&#x20;

Looking at the index.go code, we see this interesting function.

&#x20;

{width="7.6875in" height="6.59375in"}

&#x20;

So this would execute if this value of f is equals to 1.

As it turns out, deploy.sh is completely empty!

!\[\[52\_Ophiuchi\_image0020.png]]

We could do this to write authorized keys into the root folder for SSH or something.

However, this seems to take code from main.wasm.

&#x20;

Wasm is web assembly basically. I am not sure at all how to go about this portion.

In the meantime, I switched to an SSH shell for a more stable and easy shell to use.

&#x20;

Anyways this shell reads from main.wasm and executes it accordingly.

WASM is written in an assembly language, so I need to get it out of there somehow to break it down. There is a value in it, and if I can change that then it will let me execute the code within deploy.sh.

&#x20;

Transfer the files over net cat.

!\[\[52\_Ophiuchi\_image0021.png]]

&#x20;

!\[\[52\_Ophiuchi\_image0022.png]]

&#x20;

Load this thing inside some web disassembler and take a look at the code!

!\[\[52\_Ophiuchi\_image0023.png]]

We can see that there is basically one function of this entire script, which is to return 0. In basis, we need to edit this to make sure that it reflects 1 or any other value.

&#x20;

This would satisfy the condition and make it such that we are able to execute the bin script.

&#x20;

Looking at the hex code, we see this.

!\[\[52\_Ophiuchi\_image0024.png]]

The null byte there needs to be changed.

I googled around to see what I can do to change this one byte to something else.

!\[\[52\_Ophiuchi\_image0025.png]]

Found out that dd can be used to change the contents of the file, now just time to figure out how to use this properly.

!\[\[52\_Ophiuchi\_image0026.png]]

This command seems to be what I need.

So the of=test.wasm (created a test file just for this command)

Bs=1

Seek=110 as per using word count and stuff

Conv=notrunc

Count =1 for one byte!

!\[\[52\_Ophiuchi\_image0027.png]]

Time to view the result.

&#x20;

!\[\[52\_Ophiuchi\_image0028.png]]

Now this reflects 1.

Now time to change it on the machine. Take note that I forgot to append count=1 within this code.

&#x20;

Make sure to copy over the main.wasm file to another directory.

!\[\[52\_Ophiuchi\_image0029.png]]

&#x20;

This will change one byte of the entire assembly code.

Now when we run the command, this is the output.

!\[\[52\_Ophiuchi\_image0030.png]]

Sick!

Let's echo in our key to the machine so we can just SSH in.

!\[\[52\_Ophiuchi\_image0031.png]]

Run the command once more.

!\[\[52\_Ophiuchi\_image0032.png]]

SSH in!

!\[\[52\_Ophiuchi\_image0033.png]]

Pwned.

I really enjoyed this box.

&#x20;

Now I know a bit more about wasm and that I can change direct bytes of it using dd.

Also I learnt more about how I can simply transfer files using netcat.

&#x20;
