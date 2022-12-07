# Canary Bypass

## Stack Canaries

Stack canaries are used to detect a stack buffer overflow before execution of any malicious code can occur. This method works by placing a **small integer**, which value has been randomly chosen at the start of the program within the stack **just before the stack return pointer**.&#x20;

We can illustrate how it works through analysing the stack contents:

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

When the binary runs, it checks the canary and ensures that it does not change. If altered, the execution would end immediately.&#x20;

Stack canaries are checked for their value just before the return to the calling function, which is where attackers would normally gain control over the instruction pointer.&#x20;

### Types of Canaries

There are many types of canaries, but here are the most common ones:

|                   |                     |                          |
| ----------------- | ------------------- | ------------------------ |
| _Type_            | _Example_           | _Protection_             |
| Null canary       | 0x00000000          | 0x00                     |
| Terminator canary | 0x00000aff          | 0x00, 0x0a, 0xff         |
| Random canary     | \<any 4-byte value> | Usually starts with 0x00 |
| Random XOR canary |                     | Usually starts with 0x00 |
| 64-bit canary     | <8 bytes>           |                          |
| Custom canary     |                     |                          |

Each canary has their own set of advantages and security measures introduced:

* Null Canary is the most simple as it's just 4 NULL bytes. However, this is the most vulnerable of canaries because its predictable and **there are functions that can read null canaries**.
* Terminator Canaries introduces two or more hex values that attempt to terminate string operations. These values are still predictable
* Random Canaries are better in terms fo protection, and **usually consist of a NULL byte followed by 3 random bytes**. The NULL byte would attempt to terminate string operations, while the 3 random bytes will make the canary less predictable.
* The random XOR canary is similar to the random canary, but its value is XOR'd against a non-static value in the program, such as the EBP. Since most binaries run with ASLR enabled, this provides a lot more security.&#x20;

## Bypasses
