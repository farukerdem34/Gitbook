---
description: Tough enumeration...
---

# Absolute (AD) (WIP!)

This is an AD machine, so first we can begin with a port scan, and then go through the usual AD methodology for finding a weakpoint for this system.

## Gaining Access

Nmap Scan:

<figure><img src="../../../.gitbook/assets/image (20) (2).png" alt=""><figcaption></figcaption></figure>

Standard Ports. I added absolute.htb  and its DC into my hosts file for this machine, as it is standard HTB practice. There are few things to enumerate:

* Website enumeration for directories, exploits or whatever else is useful.
* DNS for hidden domains and endpoints
* SMB Shares and LDAP services that accept null credentials.
* Kerbrute to find usernames (last resort)

### Kerbrute and AS-REP

Interestingly, doing all of these revealed nothing useful, **except for my last resort.** Running a kerbrute reveals this:

<figure><img src="../../../.gitbook/assets/image (32) (1).png" alt=""><figcaption></figcaption></figure>

This username wordlist was just within my machine from another machine that required it. Very useful! However, these usernames cannot be used to do anything, leading me to believe that there are other users on this domain.

Wordlist Used:

{% embed url="https://github.com/attackdebris/kerberos_enum_userlists/blob/master/A-Z.Surnames.txt" %}

So now we have some possible users. I wanted to try and fuzz along this line, since we know that the usernames are in this format. Because this machine is rated insane, the username we need is probably not within any common wordlist.&#x20;

So to circumvent this, I took the names.txt file from Seclists (/seclists/usernames/names/names.txt) then appended the front of each entry with a letter and brute forced it. From A - Z. This would produce a list of names with the prefix required.

<figure><img src="../../../.gitbook/assets/image (8) (2) (2) (1).png" alt=""><figcaption></figcaption></figure>

Everytime I would find a username, I would then test it for AS-REP roasting and check for null credentials. Eventually, I found this d.klay user.

<figure><img src="../../../.gitbook/assets/image (31) (1).png" alt=""><figcaption></figcaption></figure>

When testing this for AS-REP Roasting, it worked!

<figure><img src="../../../.gitbook/assets/image (11) (4) (1).png" alt=""><figcaption></figcaption></figure>

Then we can crack this hash.

<figure><img src="../../../.gitbook/assets/image (3) (4) (1).png" alt=""><figcaption></figcaption></figure>

### Retrieving Tickets

It would seem that this set of credentials cannot grant us access via evil-winrm. There was a unique reason however.

<figure><img src="../../../.gitbook/assets/image (18) (4).png" alt=""><figcaption></figcaption></figure>

We can attempt with LDAP as well and get a false result. It seems that **passwords are not accepted here**. So, the next form of authentication is through tickets. Now the goal is to somehow get a ticket to authenticate into the machine. Once we get some form of ticket, we can perhaps continue with Bloodhound, login or something.

I managed to retrieve a ticket using getTGT. We can then export this.

<figure><img src="../../../.gitbook/assets/image (7) (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

We can attempt kerberoasting the machine to try and get some kind of service ticket using the credentials using GetUserSPNs. The output using the DC Domain is below:

<figure><img src="../../../.gitbook/assets/image (1) (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can fix the clock skew issue pretty easily.

```bash
sudo apt install ntpdate
sudo timedatectl set-ntp false
sudo ntpdate -s absolute.htb
```

Kerberoasting reveals that there are no SPNs to roast. Instead, we can use this ticket with CME to enumerate LDAP and SMB. This was based on the documentation of CrackMapExec, which is something I did not know prior to this.

{% embed url="https://wiki.porchetta.industries/getting-started/using-kerberos" %}

<figure><img src="../../../.gitbook/assets/image (5) (4) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (12) (4).png" alt=""><figcaption></figcaption></figure>

Really interesting output. We have found another user and credential!

### svc\_smb & Hounding

Running the same process, we can retrieve another ticket.

<figure><img src="../../../.gitbook/assets/image (29) (1) (1).png" alt=""><figcaption></figcaption></figure>

Interesting, now that we have a ticket, we can export this. I found that we can access shares from the DC using this ticket to authenticate ourselves.

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (3).png" alt=""><figcaption></figcaption></figure>

We can check out the 'Shared' share to find some interesting files.

<figure><img src="../../../.gitbook/assets/image (150).png" alt=""><figcaption></figcaption></figure>

Interesting!  The program here seems to be some form of script that creates the binary.

```bash
#!/bin/bash

nim c -d:mingw --app:gui --cc:gcc -d:danger -d:strip $1
```

Poking around the shares, we don't seem to get much from it. I could decompiled the binary, and perhaps I could find a password there.&#x20;

The next step was to use Bloodhound, since we had credentials and a ticket.

<figure><img src="../../../.gitbook/assets/image (1) (4).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://github.com/jazzpizazz/BloodHound.py-Kerberos" %}

Now we just need to fire up bloodhound and neo4j to view this data in a neat format. Bloodhound reveals a few users that are significant.&#x20;

<figure><img src="../../../.gitbook/assets/image (2) (4).png" alt=""><figcaption></figcaption></figure>

Out of all of these users, m.lovegod has the most privileges. The user owns the Network Audit group. **This group has GenericWrite over the WinRM\_User**, which I suspect is where the user flag would be. So our exploit path is clear.&#x20;

<figure><img src="../../../.gitbook/assets/image (3) (1) (2).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (33) (2).png" alt=""><figcaption></figcaption></figure>

We now need to somehow get a ticket from this m.lovegod user, or find his credentials.&#x20;

### Test.exe and Shell

When I ran the binary on my Windows VM, it seems to exit straightaway. I started Wireshark to see what I could capture from it, guessing that it was trying to make a connection back to the host somehow.

I found this interesting tidbit here.

<figure><img src="../../../.gitbook/assets/image (3) (1) (3).png" alt=""><figcaption></figcaption></figure>

Seems like there's a password being transmitted here to the LDAP server. The next step is trivial, we just need to run this thing on a VM connected to the VPN and we should get somewhere.

Doing so let me find these credentials: `absolute.htb\m.lovegod:AbsoluteLDAP2022!`

Now I was able to retrieve this user's ticket.

<figure><img src="../../../.gitbook/assets/image (1) (1) (3).png" alt=""><figcaption></figcaption></figure>

Now, perhaps we can perform some kind of actions as the user remotely. Using this ticket, we can leverage a tool called pywhisker to perform actions on the host.&#x20;

{% embed url="https://github.com/ShutdownRepo/pywhisker" %}

This is where I got stuck again...not too sure how to add the user into the group. WIP!
