# CTFs

## CTFs

Capture-The-Flags are basically computer security competitions that involve using cybersecurity skills. The most common type of CTF is Jeopardy style, where basically it consists of individual challenges. The goal of each challenge is to find a string of text known as the flag, like **flag{this\_is\_a\_fak3\_flag}.**&#x20;

The other type of CTF is called attack/defense, which is a real-life competition where teams are actively defending and attacking a network. I don't have much experience in this kind of CTF.

### Challenge Types

1. Web
   * Challenge spawns a website, and there is a web attack vector to use to gain the flag.
   * Could be stuff like **SQL Injection, OS Command Injection, Server Side Request Forgery etc.**
2. Forensics
   * Analysis of some kind of log file or disk image and the flag is contained within it.
   * Steganography is technically part of this(?), and that involves finding hidden information in images.
   * Such files include **packet captures, images, .git repositories etc.**
3. Pwn / Binary Exploitation
   * To exploit a program running on a server to find the flag.
   * Generally, they give you the program (.exe / .elf) that is running on a port of the server, and fuzzing, decompiling and reverse engineering is needed to find the vulnerability in how the program processes user information is needed.&#x20;
   * Buffer Overflow and its variances are typically used here, and this is where Python scripting is so useful (pwntools).&#x20;
4. Cryptography
   * Decrypting or encrypting a piece of data that is basically the flag.
   * Involves math and exploiting limitations of certain algorithms.&#x20;
5. Reverse Engineering
   * Self-explanatory. Given a program or file, find out how it works and RE it to find exploitable vulnerabilities.&#x20;
6. Misc.
   * Could be anything! The most wildcard of all the challenges.&#x20;
   * Open Source Intelligence Gathering (OSINT), which is basically intense Googling is sometimes  here, if not a category on its own.&#x20;

CTFs are insanely fun and one can learn a lot from doing them. It's a great way to start cybersecurity.

## Where to Start

### [https://ctftime.org/](https://ctftime.org/)

**CTF event tracker, just sign up and join one!**

### ****[**https://ctfs.github.io/resources/**](http://ctfs.github.io/resources/)****

**Learn more about common CTF techniques, but take note this is not complete.**

### [https://ctflearn.com](https://ctflearn.com/)&#x20;

**A website to practice user created CTF challenges at your own time and pace.**

### ****[**https://picoctf.com**](https://picoctf.org/)****

**CTF platform hosted by Carnegie Mellon University for everyone to learn more about security. You can either take part in the upcoming PicoCTF, or attempt challenges from the past.**

There are many more websites and stuff that have CTFs happening. &#x20;

Apart from taking part in CTFs, **do ensure to read lots of writeups as well**. Learning how other people approach challenges and the methodologies they use to solve them is crucial in getting better.&#x20;

I recommend finding a friend, or a team and tackle these challenges together.
