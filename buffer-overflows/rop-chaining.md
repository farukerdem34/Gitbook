# ROP Chaining

Return Oriented Programming (ROP) is technique that is able to bypass the NX security feature as well. **Technically, Ret2Libc is a subset of ROP**.&#x20;

ROP involves the creation of new stack frames through controlling of return addresses to **jump** to different fragments of code, called gadgets. Ret2libc is a type of ROP because we still create stack frames, however we are simply utilizing the `system()` function to gain a shell instead. &#x20;

For ROP Chaining, we jump to different fragments of code and sort of 'build' a program within the program by chaining the different fragments together. ROP is really cool because we can literally **program the code to execute what we want**, and create new routines that are not intended at all.&#x20;

## How it Works

We will be utilizing a binary that has ASLR, but NX enabled. Here's a sample of code that would be exploitable:

```c
#include <stdio.h>
#include <string.h>

void fun1(){
	printf("1\n");
}

void fun2(){
	printf("2\n");
}

void fun3(){
	printf("3\n");
}

void rop(char *string){
	char buffer[50];
	strcpy(buffer, string);
}

int main(int argc, char** argv){
	rop(argv[1]);
	return 0;
}
```

We can compile this program using the following commands:

```bash
echo 0 > /proc/sys/kernel/randomize_va_space # disable ASLR, which is enabled by default
gcc -m32 -fno-stack-protector -z execstack vulnerable.c -no-pie -o vuln
echo 1 > /proc/sys/kernel/randomize_va_space
```

We should end up with a binary with these features:

<figure><img src="../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

Most importantly, for now, we will disable ASLR to allow for easier exploitation. So our binary takes in one input and does does a `strcpy()` with it. We can check that it segfaults if given too long of an input:

<figure><img src="../.gitbook/assets/image (92).png" alt=""><figcaption></figcaption></figure>

### Code Analysis

There are 3 functions witin the code that are not meant to be executed in anyway. The program starts at `main()`, which calls the `rop()` function to execute a `strcpy(buffer, string)` instruction. We already know that `strcpy()` is vulnerable to buffer overflow because it does not check for length.&#x20;

The goal here is to trigger **all 3 of these functions using basic ROP chaining in numerical order**. First, we know to overflow the `strcpy()` function, which I want to use to call `fun1()`. Then, `fun1()` would return and call `fun2()`, and this repeats until all are called, then I would call `exit()` to let the program quit.

Our payload would look like this:

```
payload = 50 "A" + BBBB  + &fun1 + &fun2 + &fun3 + &exit
```

Now, we need to open this up in `gdb` to analyse its contents.

```bash
gdb rop
b main
r
```

Then, we can find the addresses of these functions.

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

My addresses are static because ASLR is disabled, so no worries for that. Now, we need to find the offset needed. The offset should be about 50, but I'll generate a pattern of length 70 in case.

<figure><img src="../.gitbook/assets/image (90).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

The offset is 62, and now we can start to construct our exploit script.&#x20;
