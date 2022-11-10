# Exghost

Exghost

Wednesday, 30 March 2022

5:00 pm

A Linux machine with HTTP and FTP open. I brute forced the HTTP login and eventually found some credentials.

{width="7.375in" height="0.8229166666666666in"}

When logged in, we get one file here.

!\[\[43\_Exghost \_image002.png]]

Looking through this file, we can see that it's some kind of pcap file here. We can follow the requests, but we still only get one request out of this.

&#x20;

There are a few HTTP requests here and there, but nothing of interest.

&#x20;

!\[\[43\_Exghost \_image003.png]]

We can confirm these directories to exist.

&#x20;

!\[\[43\_Exghost \_image004.png]]

&#x20;

From here, we can actually find out that this backup file is a Pcapng file, so let's wireshark it.

&#x20;

From the objects that we can export form this file, we can get this little thing:

!\[\[43\_Exghost \_image005.png]]

&#x20;

From here, it seems that we are able to send POST requests to u

pload data on this file.

We can craft one.

!\[\[43\_Exghost \_image006.png]]

&#x20;

A gobuster also reveals that there is an uploads folder here, and we can try uploading some kind of PHP web shell and get RCE on the machine using this method.

&#x20;

We can also see the file that was uploaded if we need to.

!\[\[43\_Exghost \_image007.png]]

&#x20;

This would give us a hint about what was actually uploaded onto the device.

Now, we know that there is definitely some form of HTML uploads folder there.

After a bit of tinkering, I was able to come up with this:

!\[\[43\_Exghost \_image008.png]]

&#x20;

So we need to ensure that the file has the correct thing, yet we need to achieve RCE. This can be done easily using exiftool to embed a PHP shell within it.

Seems that the name of the file has to stay as testme and myFile for it to continue accepting the file.

&#x20;

Turns out this version of the exiftool that is found within the packet capture is vulnerable to RCE. From there, we can use Curl and the -F flag to upload files onto the device and be able to gain a reverse shell. After shelling it, we can use the common Sudo exploit in order to finish the exploit and become root.

&#x20;

Pretty simple box besides the brute forcing.

&#x20;
