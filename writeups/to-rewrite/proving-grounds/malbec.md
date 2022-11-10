# Malbec

Malbec

Sunday, July 3, 2022

1:43 PM

Nmap scan:

!\[\[69\_Malbec\_image001.png]]

&#x20;

All we get is this.

{width="6.895833333333333in" height="3.5416666666666665in"}

&#x20;

We can see that there is an exe file. I'm guessing we need to either BOF or RE this.

Right, so we can download this binary and move it to our Windows machine.

&#x20;

{width="6.5625in" height="1.375in"}

&#x20;

We can see that the exe file listens on port 7138.

{width="5.40625in" height="0.4895833333333333in"}

&#x20;

And then we can see how the thing is able to basically load stuff...?

{width="8.947916666666666in" height="2.2291666666666665in"}

&#x20;

We can try to generate a buffer length.

{width="9.75in" height="1.9375in"}

&#x20;

{width="9.791666666666666in" height="2.0104166666666665in"}

&#x20;

{width="6.708333333333333in" height="1.03125in"}

&#x20;

So the program crashes, which leads me to believe this is a classic BOF.

&#x20;

We can find the offset easily.

{width="4.34375in" height="0.8854166666666666in"}

&#x20;

Now, we need to fuzz bad characters.

&#x20;

Create a bytearray:

!\[\[69\_Malbec\_image0010.png]]

&#x20;

Then we can add the badchars to our script.

!\[\[69\_Malbec\_image0011.png]]

&#x20;

Send it again.

&#x20;

!\[\[69\_Malbec\_image0012.png]]

&#x20;

There are no bad chars...?

Interesting.

&#x20;

Now let's think about how to gain a shell from this.

It's a linux machine hosting this exe, so just make sure to use a linux based payload and it should be fine.

&#x20;

{width="9.229166666666666in" height="3.2604166666666665in"}

&#x20;

!\[\[69\_Malbec\_image0014.png]]

&#x20;

{width="7.5625in" height="1.5833333333333333in"}

&#x20;

!\[\[69\_Malbec\_image0016.png]]

&#x20;

&#x20;

PE:

Running linpeas, we can see what stuff lies within this machine.

!\[\[69\_Malbec\_image0017.png]]

&#x20;

We have root running ldconfig here, although I do not think this is anything as of yet.

&#x20;

!\[\[69\_Malbec\_image0018.png]]

&#x20;

It seems that we can make use of ldso to do our privilege escalation.

Malbec runs with the configuration of this.

&#x20;

Lastly, the most notable one is this new SUID.

!\[\[69\_Malbec\_image0019.png]]

&#x20;

!\[\[69\_Malbec\_image0020.png]]

&#x20;

We can check ldd.

!\[\[69\_Malbec\_image0021.png]]

&#x20;

This was unique because it looks for this file, which has not been put anywhere, and in this case, we can probably set it such that it would find the binary at our own location, and perhaps replace the binary with some malicious .so file that would give us a reverse shell or spawn a root shell. Since the file was called libmalbec.so, I assume that the malbec.conf that had the user's home directory meant that all we had to do was simply put a malicious library within our own directory and then execute the binary.

&#x20;

Program to be compiled:

!\[\[69\_Malbec\_image0022.png]]

&#x20;

{width="9.697916666666666in" height="1.8125in"}

&#x20;

Then place inside the home directory of carlos and just run messenger. (after renaming it appropriately)

!\[\[69\_Malbec\_image0024.png]]

&#x20;

Flag:

!\[\[69\_Malbec\_image0025.png]]

4706990b1bdcedc8a7e691d2a7813302
