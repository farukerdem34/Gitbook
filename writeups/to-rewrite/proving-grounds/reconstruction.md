# Reconstruction

Reconstruction

Saturday, July 9, 2022

12:43 AM

Nmap scan:

!\[\[67\_Reconstruction\_image001.png]]

&#x20;

!\[\[67\_Reconstruction\_image002.png]]

&#x20;

!\[\[67\_Reconstruction\_image003.png]]

&#x20;

FTP:

Anonymous access allowed, and also there are pcap files within this machine.

!\[\[67\_Reconstruction\_image004.png]]

'

We can wireshark this.

&#x20;

Wireshark:

The pcap files seem to be testing the query parameter on the main page.

!\[\[67\_Reconstruction\_image005.png]]

&#x20;

There's a login portion for this, and this works.

Right, so we have this and then the note.txt tells us this.

{width="9.479166666666666in" height="1.4375in"}

&#x20;

There should be some form of password within the pcap files somewhere, else they wouldn't suggest this.

&#x20;

I looked through the pcap files and then filtered the HTTP objects, looking for POST requests.

!\[\[67\_Reconstruction\_image007.png]]

&#x20;

I noticed one request which had a return, and went to 13745.

!\[\[67\_Reconstruction\_image008.png]]

&#x20;

This password works.

&#x20;

Port 8080:

This is a werkzeug server, meaning there is some form of console, but this one is pin-protected.

!\[\[67\_Reconstruction\_image009.png]]

&#x20;

Anyways, with the password found earlier, we can login.

&#x20;

!\[\[67\_Reconstruction\_image0010.png]]

&#x20;

With this, we can create new blog entries or something.

Interestingly, there is also this JWT token when accessing the pages.

&#x20;

This tells me this may be a flask application. In which case, we could potentially get our the secret or something using this token.

However, even when logging in as the page does not allow us to create posts, and we are always referred to the debugger page.

&#x20;

We should be trying to find that WebSOC endpoint and fuzzing the server.

&#x20;

Wfuzz:

!\[\[67\_Reconstruction\_image0011.png]]

&#x20;

!\[\[67\_Reconstruction\_image0012.png]]

&#x20;

Data and create.

&#x20;

/data:

!\[\[67\_Reconstruction\_image0013.png]]

&#x20;

We can fuzz this data parameter further using the same command.

And we would notice this X-Error header here.

!\[\[67\_Reconstruction\_image0014.png]]

&#x20;

Incorrect padding...?

This looks like some kind of weird b64 error.

&#x20;

So I tried to include some base64 file names, and behold, we have an LFI.

&#x20;

!\[\[67\_Reconstruction\_image0015.png]]

&#x20;

This combined with the fact it is possible to reconstruct the console PIN using some local file parameters means we almost have RCE.

&#x20;

We need a few things from this.

&#x20;

Username which is jack

!\[\[67\_Reconstruction\_image0016.png]]

> &#x20;

We need the modname, which would be within the /proc/self/cgroup file.

!\[\[67\_Reconstruction\_image0017.png]]

&#x20;

&#x20;

The localtion of the file, which can be found from the console output

!\[\[67\_Reconstruction\_image0018.png]]

&#x20;

Then we need to find the machine-id through /etc/machine-id.

!\[\[67\_Reconstruction\_image0019.png]]

&#x20;

Afterwards, it would be the MAC address of the device, converted into decimal expression.

&#x20;

To find the MAC address, we would first need to find the ARP cache from /proc/net/arp

!\[\[67\_Reconstruction\_image0020.png]]

&#x20;

Afterwards, we can find the MAC address from /sys/class/net/ens160/address.

&#x20;

!\[\[67\_Reconstruction\_image0021.png]]

&#x20;

Then convert this to decimal:

!\[\[67\_Reconstruction\_image0022.png]]

&#x20;

Then we can grab a script from here once we have these parameters.

[https://gist.github.com/InfoSecJack/70033ecb7dde4195661a1f6ed7990d42](https://gist.github.com/InfoSecJack/70033ecb7dde4195661a1f6ed7990d42)

&#x20;

Run the script and here we go.

{width="8.947916666666666in" height="3.0729166666666665in"}

(my ARP changed cus I restarted the box)

&#x20;

Based on hacktricks, sometimes we may have to append the blog.service to the machineid.

!\[\[67\_Reconstruction\_image0024.png]]

&#x20;

{width="9.822916666666666in" height="3.1354166666666665in"}

&#x20;

Just try both and we would be able to figure something out regarding this.

&#x20;

From there, we can easily gain a reverse shell.

{width="5.708333333333333in" height="1.3958333333333333in"}

&#x20;

Grab a shell from revshells.com python.

&#x20;

{width="7.583333333333333in" height="2.03125in"}

&#x20;

Within the app.py, we can find an old password.

!\[\[67\_Reconstruction\_image0028.png]]

&#x20;

With this password, we can su to jack.

!\[\[67\_Reconstruction\_image0029.png]]

&#x20;

Flag:

!\[\[67\_Reconstruction\_image0030.png]]

c9578f61b8bef402e9a5844d3984eebb

&#x20;

PE:

Jack has powershell history for some reason.

{width="5.75in" height="2.0625in"}

&#x20;

There's a password here.

With this, we can su to root.

&#x20;

!\[\[67\_Reconstruction\_image0032.png]]

&#x20;

Flag:

!\[\[67\_Reconstruction\_image0033.png]]

&#x20;

&#x20;
