# Servmon (Unstable box)

Servmon (Unstable box)

Thursday, 17 February 2022

10:14 pm

This is a Windows machine.

The target IP is 10.10.10.184.

My IP is 10.10.16.9.

&#x20;

Enumeration first up.

!\[\[43\_Servmon (Unstable box)\_image001.png]]

Aight, these ports. I ran two scans, one for the SMB ports and one more vuln nmap scan to check on what I can do.

{width="11.041666666666666in" height="2.5416666666666665in"}

Enum4Linux did not work.

And the nmap scan did not reveal anything.

&#x20;

I tried an anonymous FTP login after that, because sometimes it works.

&#x20;

!\[\[43\_Servmon (Unstable box)\_image003.png]]

Well. There is one Users file open on this, and we can investigate further using this.

There were two files within these directories.

!\[\[43\_Servmon (Unstable box)\_image004.png]]

&#x20;

Transferred both .txt files to my machine and let's view what we got.

!\[\[43\_Servmon (Unstable box)\_image005.png]]

&#x20;

!\[\[43\_Servmon (Unstable box)\_image006.png]]

So there is a passwords.txt file on the Desktop somewhere... Seems that all the SMB ports are let downs.

&#x20;

The port 8443 is a HTTP port, and we are allowed to connect to that one.

!\[\[43\_Servmon (Unstable box)\_image007.png]]

After referring to the solutions and doing a full port scan, it was revealed there was a port 80...

&#x20;

!\[\[43\_Servmon (Unstable box)\_image008.png]]

That was pretty annoying, I still can't find it. I resetted the box and tried again. I could not even connect to the damn thing. Took a long while before I connected, no wonder this box is rated so lowly.

&#x20;

To save the time, I determined that I already knew where this password file exists.

&#x20;

There was a directory traversal attack that could have been used. This would display the possible passwords. One of them would lead to being able to use SSH.

&#x20;

This is because the web browser on port 80 would display NVMS-1000, which is vulnerable to a directory traversal attack.

!\[\[43\_Servmon (Unstable box)\_image009.png]]

&#x20;

We know that this password.txt file is on the Desktop of Nathan, so just use this to get into it.

&#x20;

Anyways let's continue from here.

The other hint from the ftp I found included the use of NVMS...

Unfortunately, the box did not allow me to connect, and I skipped it because it was just too unstable.
