# Worker

Worker

Friday, 11 March 2022

10:49 pm

This is a Windows machine.

The target IP is 10.10.10.203.

My IP is 10.10.16.9.

&#x20;

Enumeration time.

&#x20;

!\[\[78\_Worker\_image001.png]]

&#x20;

!\[\[78\_Worker\_image002.png]]

Aight. The regular port 80 is just another IIS server.

Port 5985 is a 404.

&#x20;

Port 3690 is a subversion server, and I do believe there are some interesting things that can be found on that server.

Also, I turned on Burpsuite to check my requests and where they are going. Worker.htb has been added to the hosts file too.

&#x20;

Going onto hacktricks, it reveals that we are able to connect to this svn server.

!\[\[78\_Worker\_image003.png]]

We got another domain name, and we have this txt file. I added this to our hosts file too.

&#x20;

Website:

!\[\[78\_Worker\_image004.png]]

&#x20;

Aight, some website we have here.

Looking at the logs of the server, we have a user named nathen.

!\[\[78\_Worker\_image005.png]]

&#x20;

We can download the whole repository as well, and view the different versions that are present on it.

!\[\[78\_Worker\_image006.png]]

&#x20;

From here, we can view the different versions that are present on this repository.

{width="4.15625in" height="3.8229166666666665in"}

There's this deploy.ps1 present, and we can perhaps do something with it.

!\[\[78\_Worker\_image008.png]]

Right, so we have a username and password here. There's also this copy-site.ps1, presumably to move the website elsewhere.

&#x20;

There's probably other forms of websites present here, so I did a wfuzz. There were some subdomains present on the machine. Most importantly is the devops one.

&#x20;

The moved.txt mentioned that it moved to a devops server...So...

!\[\[78\_Worker\_image009.png]]

&#x20;

Upon visit, it prompts us with some creds:

!\[\[78\_Worker\_image0010.png]]

I tried the nathen one, and it worked!

!\[\[78\_Worker\_image0011.png]]

Azure DevOps server...Hmm...

&#x20;

Looked at this repository, and explored a little bit.

Looking at the repos, it seems that only spectral is active.

&#x20;

Perhaps this is another website we need to visit. I added spectral.worker.htb to my hosts file.

This works!

!\[\[78\_Worker\_image0012.png]]

I tried uploading a reverse shell to the repository with the aspx and asp extensions, because I knew that Microsoft IIS servers mainly use asp/aspx shells.

{width="5.083333333333333in" height="5.770833333333333in"}

We need to create a pull request for this:

{width="5.302083333333333in" height="6.71875in"}

Then approve the pull request:

!\[\[78\_Worker\_image0015.png]]

Turns out we need to indicate a work item by ID, which I forgot. Oops.

&#x20;

!\[\[78\_Worker\_image0016.png]]

&#x20;

!\[\[78\_Worker\_image0017.png]]

&#x20;

Afterwards, we should see this:

!\[\[78\_Worker\_image0018.png]]

I redid the same thing for another repository, this time for alpha and it worked!

I created a new branch, then uploaded my file, approved my request and then merged the branches.

Activated the shell like so:

!\[\[78\_Worker\_image0019.png]]

&#x20;

!\[\[78\_Worker\_image0020.png]]

&#x20;

!\[\[78\_Worker\_image0021.png]]

Aight, we're this guy.

&#x20;

Checked around, and found nothing of interest.

This is a cloud server, seeing as that there are Azure. I remember managing google drive, and this had a different drive by the name of G:/

&#x20;

I ran this command to check, and sure enough there was another drive:

!\[\[78\_Worker\_image0022.png]]

&#x20;

!\[\[78\_Worker\_image0023.png]]

Change directories and we can see the svnrepos, which is something interesting.

&#x20;

There was a conf directory within it with some interesting stuff.

!\[\[78\_Worker\_image0024.png]]

This passwd file was the most interesting.

{width="3.1875in" height="9.197916666666666in"}

Had the password of every user here.

&#x20;

Coped all of them.

&#x20;

Within the C drive user file, there was a robisl.

!\[\[78\_Worker\_image0026.png]]

Before anything, I tried the evil-winrm trick, and it worked with the password of the user.

&#x20;

!\[\[78\_Worker\_image0027.png]]

Grab the user flag, and let's continue on.

Downloaded winpeas64 onto the machine and ran it.

&#x20;

Here we can see that the user is part of Remote Management Users, which allows for the Evil-WinRM to work:

!\[\[78\_Worker\_image0028.png]]

&#x20;

There was nothing much here, so with the credentials I tried logging in as a different user on the website.

I could sign in:

!\[\[78\_Worker\_image0029.png]]

&#x20;

From here, I looked at the one project that was open.

Googling around, I found [this](https://devblogs.microsoft.com/devops/pipeline-stealing-another-repo/) website which showed us how to use pipelines that would get us files that we weren't able to see.

Turns out editing pipelines is really powerful, and could be the key to PE maybe.

&#x20;

Found this as well:

!\[\[78\_Worker\_image0030.png]]

So anyways, I was able to create new pipelines through YAML files.

!\[\[78\_Worker\_image0031.png]]

This gave me the idea of creating a reverse shell or something perhaps.

From here, I looked at the pipelines that I could create

!\[\[78\_Worker\_image0032.png]]

Starter pipelines looks the simplest amongst all of them.

&#x20;

I selected it and was dropped into a script:

!\[\[78\_Worker\_image0033.png]]

Tried it and it failed saying that I do not have a Default pool, which means I am not authorized to use it. I deleted the line and tried again with this:

!\[\[78\_Worker\_image0034.png]]

&#x20;

!\[\[78\_Worker\_image0035.png]]

This worked...somewhat.

!\[\[78\_Worker\_image0036.png]]

It connected but didn't really give me what I wanted.

Perhaps I need to find another pool instead.

&#x20;

Looked at the pools and found there was only one I could use:

!\[\[78\_Worker\_image0037.png]]

Perhaps let's try this one.

!\[\[78\_Worker\_image0038.png]]

Changed the port too, because it seems like 443 is the safest here.

&#x20;

Did the same things again.

Eventually got a shell!

!\[\[78\_Worker\_image0039.png]]

Pwned.

Love the implementation of Azure DevOps here.

&#x20;

&#x20;

&#x20;
