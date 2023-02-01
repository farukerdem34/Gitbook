# ASLR Bypass

Address Space Layout Randomisation (ASLR) is a feature that causes memory addresses of functions and instructions to be randomised. Each time we run a binary, all of the addresses would change and never be the same.

In earlier Buffer Overflows, we examined how controlling the EIP can lead to RCE, with or without NX protection. However, with ASLR enabled, even if we can control the EIP, we cannot 'jump' anywhere because we wouldn't know where to jump to with ASLR enabled.

## Concepts

In order to bypass ASLR, we need to understand how it functions, as well as how functions are called. When we run a binary, the libraries and functions of that binary are loaded into virtual memory.

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

With ASLR enabled, the library would be **loaded at different places in memory** each time. With the main library being loaded differently, all functions called in the library are affected and have different locations in memory.

Functions are called based on **offsets**. For example, if the libc library is loaded at `0x10000000`, and the offset for the `printf()` function is `0x00001000`, then when a program is run and `printf()` is called, it is mapped at `0x10001000`. \
Generally, base address (where library is called) + offset = memory location of function.

The vulnerability arises because when ASLR is enabled, **the offset does not change and is constant**. So, if we are able to find the base address where the library is called, we can use the constant offsets to load certain functions. ****&#x20;

### **Methods**

To bypass ASLR, there are a few methods possible

* Information Leak Vulnerability
  * Can be an LFI or anything else that lets us **read memory locations on the machine**.&#x20;
  * Memory disclosure vulnerabilities also can work.
* Brute Forcing
  * Perhaps the range of addresses where the library is loaded is rather small. This indicates that the base address could be brute forced and a simple for loop can cover all of it rather quickly.&#x20;
