# Binary Security

Binaries, like .exe or .elf files, have security implementations that we need to know how to enumerate and understand. While there are some security mechanisms present, this does not mean its unexploitable, just a lot harder.

## ASLR

Address Space Layout Randomization (ASLR) is a measure that is meant to introduce randomness of executables, the libraries it uses and the stack in memory address space.

For example, take a look at this snippet:

```c
#include <stdlib.h>
int main(){
    system("echo 'hello world!'");
    return 0;
}
```

This is a program that basically executes a `system` function, which allows us to execute any command we want as if we were in a command terminal. Each time a program uses a library, it would call that library to find the pre-defined functions that it needs, in this case `system` from `stdlib.h`.  This `system` function lives within the `stdlib.h`, and has a specific address of which it exists. This address is specified as an offset.&#x20;

For example if the base address is loaded at `0x12345678`, and the offset is `0x00000001`, then the `system` function would be at `0x12345677` within the library on our machine.&#x20;

Without ASLR, the address of which this function exists in is **always the same**. The memory address is static, and this can be rather dangerous as predictability can allow for buffer overflows (which rely on memory addresses) to be rather easy.

With ASLR, the OS loads the same executable at **different locations in memory each time**. This means that everytime we run this program, the memory addresses are completely different. This would mean that the function effectively 'lives' at a different area as well.&#x20;

Binaries without this would be vulnerable to an **ASLR bypass attack.**&#x20;

## DEP

Data Execution Prevention (DEP) is a defensive hardware and software measure that **prevents the exeuction of code within memory**. This would mean that when we do an exploit on this, we cannot **inject malicious shellcode into memory** because it would not run.

There are still however, bypasses for this.

## Canary / Stack Cookie

This measure would place a value next to the return address on the stack. This prefix would prevent for the attackers from controlling the EIP and returning to wherever they want.&#x20;

The function prologue would load this value into its location, and everytime there is a return statement, the program checks to see if this value has been edited in any way. If it has, the program does not continue with execution. If nothing is detected, then it will continue as per normal

## Compiler Flags

To compile binaries with these security measures, we have to include **specific flags** when compiling. Below is a list of flags for the `gcc` compiler.&#x20;

```
GCC Security related flags and options:

CFLAGS="-fPIE -fstack-protector-all -D_FORTIFY_SOURCE=2" 
LDFLAGS="-Wl,-z,now -Wl,-z,relro"
  Hardened gentoo default flags.
  
-Wall -Wextra
  Turn on all warnings.

-Wconversion -Wsign-conversion
  Warn on unsign/sign conversions.

-Wformat­security
  Warn about uses of format functions that represent possible security problems

-Werror
  Turns all warnings into errors.

-arch x86_64
  Compile for 64-bit to take max advantage of address space (important for ASLR; more virtual address space to chose from when randomising layout).

-fstack-protector-all -Wstack-protector --param ssp-buffer-size=4
  Your choice of "-fstack-protector" does not protect all functions (see comments). You need -fstack-protector-all to guarantee guards are applied to all functions, although this will likely incur a performance penalty. Consider -fstack-protector-strong as a middle ground.
  The -Wstack-protector flag here gives warnings for any functions that aren't going to get protected.

-pie -fPIE
  For ASLR

-ftrapv
  Generates traps for signed overflow (currently bugged in gcc)

-­D_FORTIFY_SOURCE=2 ­O2
  Buffer overflow checks. See also difference between =2 and =1

­-Wl,-z,relro,-z,now
  RELRO (read-only relocation). The options relro & now specified together are known as "Full RELRO". You can specify "Partial RELRO" by omitting the now flag. RELRO marks various ELF memory sections read­only (E.g. the GOT)

If compiling on Windows, please Visual Studio instead of GCC, as some protections for Windows (ex. SEHOP) are not part of GCC, but if you must use GCC:

-Wl,dynamicbase
  Tell linker to use ASLR protection

-Wl,nxcompat
  Tell linker to use DEP protection
  
```

## Enumeration

When we first get a binary, we can use `checksec` from gdb to see the security measures that have been enabled for .elf files.

For Windows files, we can use this tool:

{% embed url="https://github.com/trailofbits/winchecksec" %}

When we get the output from the command, it would look something like this:

<figure><img src="../../.gitbook/assets/image (11) (5).png" alt=""><figcaption><p><em>Taken from HTB Retired</em></p></figcaption></figure>

We can breakdown the output from this:

* CANARY has been **disabled**, meaning there are no stack cookies present
* FORTIFY has been **disabled**, meaning there are no checks for buffer overflows within the binary (indicating this binary is vulnerable!)
* NX has been **enabled**, meaning that **no execute is enabled**. This means the memory within the stack is non-executable and DEP has been enabled. Thus, no shellcode injection here.
* PIE has been **enabled**, meaning ASLR is enabled.
* RELRO is on **FULL,** meaning that the binary is **fully read-only** and its contents cannot be edited throughout the execution of it.

With this knowledge, we would need to use a debugger to see the application better. For this machine in HTB, it was vulnerable to ROP chaning leading to RCE.&#x20;
