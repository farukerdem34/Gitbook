# Symbolic

Symbolic

Sunday, July 3, 2022

10:26 AM

Nmap scan:

!\[\[112\_Symbolic\_image001.png]]

&#x20;

!\[\[112\_Symbolic\_image002.png]]

&#x20;

Port 80:

!\[\[112\_Symbolic\_image003.png]]

&#x20;

This can be used for RFI, but it seems to take a web page and convert them to a PDF. Not sure how it does that.

But from here, we can actually use this to read files in Linux, using the file:/// wrapper.

&#x20;

First, we enumerate this web page. We should try to form a PDF and then exiftool to analyse the file.

There was SSH open on this windows machine, so perhaps looking at SSH keys was not off the table.

&#x20;

When sending stuff, we can see that it uses a PHP backend.

!\[\[112\_Symbolic\_image004.png]]

&#x20;

From here, it seems that the website is unable to process anything other than HTML files.

!\[\[112\_Symbolic\_image005.png]]

&#x20;

It would host this.

!\[\[112\_Symbolic\_image006.png]]

&#x20;

Interesting.

Gobuster did not reveal anything much regarding this application, apart from a /pdf and /log file.

&#x20;

!\[\[112\_Symbolic\_image007.png]]

&#x20;

The PDFs folder appears to store all the files for the web page.

With this HTML files, we can point towards our own web page and execute some form of LFI. However, we would need t obegin searching for methods to include php file wrappers on Windows machines.

&#x20;

The method of pointing towards a PHP file to execute commands remotely does not work. However, perhaps we can attach a file such that it would be on the page itself.

&#x20;

The usage of responder to intercept stuff does not work.

So let's think about this.

Here are the facts:

* Wkhtmltopdf is known to a lot of LFI and SSRF
* There is no method of which we can use this LFI to our advantage, unless we are to guess the different pages that are present within this.
* The page uses PHP, however, responses to PHP Web shells do not work out well, as the page is not executing the code (I imagine it does not execute anything)
*   Is there a way of which we can either:

    a. Make the machine download the files we want onto the machine?

    b. Upload a PDF that would be able to capture NTLM hashes as they are passed through SSRF?

    c. Perhaps there is indeed a config file (although unlikely, as there are no mentions of any config.php files being found within the directories...But we can indeed view the LFI stuff...?

&#x20;

Let's try C, as that's the most common exploit that is being used. We know that iframes do not work, so let's try other hooks.

&#x20;

So would basically have to leverage XSS to do this. First thing we can do is just test some basic stuff like this.

!\[\[112\_Symbolic\_image008.png]]

&#x20;

!\[\[112\_Symbolic\_image009.png]]

&#x20;

That's a start.

Now that we know making some JS to load works on this, we would need to basically be able to write some JS to read files on the websites.

&#x20;

Common stuff used is [file:///etc/passwd](file://etc/passwd), however, this would not work on our windows device.

We can see that the document is shown to be from our own server, hence we would probably need to adopt the same syntax.

!\[\[112\_Symbolic\_image0010.png]]

&#x20;

!\[\[112\_Symbolic\_image0011.png]]

&#x20;

!\[\[112\_Symbolic\_image0012.png]]

&#x20;

Finally I made it work after lotsw of googling and checking around.

!\[\[112\_Symbolic\_image0013.png]]

&#x20;

!\[\[112\_Symbolic\_image0014.png]]

&#x20;

Great!

Now we have LFI, we can proceed to enumerate the windows file system.

&#x20;

WE can read the process.php thing to find out how it works.

{width="6.8125in" height="5.0in"}

&#x20;

Perhaps the URL parameter is the vulnerable one, because no matter what, it is executed.

&#x20;

We can perhaps configure an RCE to inject inside that.

With this, we can actually capture an NTLM hash to find a user.

&#x20;

We just need to finish the command, and then append a && to have a Command Injection point.

!\[\[112\_Symbolic\_image0016.png]]

&#x20;

{width="9.760416666666666in" height="1.8125in"}

&#x20;

Now that we have a hash, we can crack it.

Either that, or we can read his SSH key.

!\[\[112\_Symbolic\_image0018.png]]

&#x20;

!\[\[112\_Symbolic\_image0019.png]]

&#x20;

{width="8.604166666666666in" height="3.0104166666666665in"}

&#x20;

We have a shell.

&#x20;

Flag:

!\[\[112\_Symbolic\_image0021.png]]

&#x20;

PE:

Within the main C:\ drive, there's this backup file.

!\[\[112\_Symbolic\_image0022.png]]

&#x20;

!\[\[112\_Symbolic\_image0023.png]]

&#x20;

We can see that what it does is basically, keep repeating something.

We can't edit this file in anyway.

&#x20;

When we see our privileges, we can see this new one.

!\[\[112\_Symbolic\_image0024.png]]

&#x20;

What this token means is that we can use KERB\_S4U\_LOGON to gain some impersonation of the administrator, without credentials, and then be able to execute commands as the administrator.

Not sure how to exploit this however.

&#x20;

!\[\[112\_Symbolic\_image0025.png]]

&#x20;

This basically means that we can run programs under the SYSTEM user. Perhaps we need to find custom EXEs

[https://github.com/gtworek/PSBits/tree/master/VirtualAccounts](https://github.com/gtworek/PSBits/tree/master/VirtualAccounts)

This repository should serve us well.

&#x20;

{width="6.3125in" height="4.083333333333333in"}

&#x20;

This spawns a weird shell...

But essentially, we do not have any output anymore, and neither can we execute any code it appears.

Weird.

&#x20;

Moving back to the backup.ps1 file, we can see how we can manipulate this.

!\[\[112\_Symbolic\_image0027.png]]

&#x20;

Firstly, we can edit the log file, meaning that we can potentially place our own symlink in this to point towards anywhere we want.

However, we cannot create any symlink without the privilege.

&#x20;

Let's view the privileges of this file that we are in.

!\[\[112\_Symbolic\_image0028.png]]

&#x20;

And from here, we would have to delete this entire file and put the symlink in somewhere that we can control.

!\[\[112\_Symbolic\_image0029.png]]

&#x20;

Then we can use CreateSymlink.exe to bypass the need for a privilege.

!\[\[112\_Symbolic\_image0030.png]]

&#x20;

Then in a second shell, we just have to execute that backup folder.

We should get the SSH key of the administrator.

!\[\[112\_Symbolic\_image0031.png]]

&#x20;

Then we can just SSH in as the administrator.

{width="6.302083333333333in" height="2.34375in"}

&#x20;

Flag:

!\[\[112\_Symbolic\_image0033.png]]

&#x20;
