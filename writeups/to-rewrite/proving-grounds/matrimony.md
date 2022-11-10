# Matrimony

Matrimony

Monday, May 30, 2022

8:51 AM

Nmap scan:

!\[\[54\_Matrimony\_image001.png]]

&#x20;

!\[\[54\_Matrimony\_image002.png]]

&#x20;

Port 53 is open on this...

&#x20;

Port 80:

!\[\[54\_Matrimony\_image003.png]]

&#x20;

!\[\[54\_Matrimony\_image004.png]]

&#x20;

!\[\[54\_Matrimony\_image005.png]]

&#x20;

Interesting, this may be a RFI exploit here.

We can test this theory out:

!\[\[54\_Matrimony\_image006.png]]

&#x20;

!\[\[54\_Matrimony\_image007.png]]

&#x20;

This works when doing the RFI, as I got a hit on my own web server. Perhaps we could upload some form of shell in this manner.

!\[\[54\_Matrimony\_image008.png]]

&#x20;

This was pointless, however.

&#x20;

Port 53:

When doing a reverse lookup, we can get a reponse:

!\[\[54\_Matrimony\_image009.png]]

&#x20;

After guessing a few domains, we can find this one that works:

!\[\[54\_Matrimony\_image0010.png]]

&#x20;

When trying a zone transfer, we can see that there is another subdomain here.

{width="9.541666666666666in" height="2.90625in"}

&#x20;

This brings us to a different website.

!\[\[54\_Matrimony\_image0012.png]]

&#x20;

Interesting because there are exploits for this website.

!\[\[54\_Matrimony\_image0013.png]]

&#x20;

Particularly, an RCE.

!\[\[54\_Matrimony\_image0014.png]]

&#x20;

!\[\[54\_Matrimony\_image0015.png]]

&#x20;

Interesting.

We can create some fake profile and use it here.

!\[\[54\_Matrimony\_image0016.png]]

&#x20;

From here, we can gain a reverse shell.

!\[\[54\_Matrimony\_image0017.png]]

&#x20;

!\[\[54\_Matrimony\_image0018.png]]

&#x20;

!\[\[54\_Matrimony\_image0019.png]]

&#x20;

Flag:

!\[\[54\_Matrimony\_image0020.png]]

&#x20;

PE:

While we're in the user directory, we can grab the private key.

!\[\[54\_Matrimony\_image0021.png]]

&#x20;

The fact there's this here, is suspicious on its own.

&#x20;

Within /opt, there's a cron.sh thing here.

!\[\[54\_Matrimony\_image0022.png]]

&#x20;

!\[\[54\_Matrimony\_image0023.png]]

&#x20;

Docker being part of this tells me there's some container within the machine.

!\[\[54\_Matrimony\_image0024.png]]

&#x20;

We can SSH further into the machine.

As the root user part of this little container, we have access to everything. When running linpeas on this, we can find this docker.sock thing.

!\[\[54\_Matrimony\_image0025.png]]

&#x20;

Interestingly, we need to download the docker binary to this machine.

!\[\[54\_Matrimony\_image0026.png]]

&#x20;

After downloading this, we can find a way to remount back to the main OS as the root user.

!\[\[54\_Matrimony\_image0027.png]]

&#x20;

From here, we can run this command to gain access into the main machine's root directory.

!\[\[54\_Matrimony\_image0028.png]]

&#x20;

Once we are here, we can go ahead and echo in our own key into the authorized\_keys folder.

&#x20;

!\[\[54\_Matrimony\_image0029.png]]

&#x20;

Then, we can just SSH in as the root user!

{width="4.34375in" height="1.0208333333333333in"}

&#x20;

This would eventually work, but I'm lazy so I grabbed the flag.

&#x20;
