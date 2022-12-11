# BountyHunter

## Gaining Access

An Nmap scan reveals that ports 22 and 80 are open.&#x20;

<figure><img src="../../../.gitbook/assets/image (204).png" alt=""><figcaption></figcaption></figure>

### Portal Endpoint

From the main webpage, we can see that there was a portal endpoint on it.

<figure><img src="../../../.gitbook/assets/image (202).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

Going to this page reveals a Beta Report System that is in use.

<figure><img src="../../../.gitbook/assets/image (176).png" alt=""><figcaption></figcaption></figure>

Trying out some random input reveals this:

<figure><img src="../../../.gitbook/assets/image (169).png" alt=""><figcaption></figcaption></figure>

When proxied through Burp, we can view that it sends a `data` parameter with a Base64 encoded XML input.

<figure><img src="../../../.gitbook/assets/image (198).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (164).png" alt=""><figcaption></figcaption></figure>

XXE injection is clearly the next stage here.

### XXE Injection for LFI

We can easily exploit this to start reading files on the webserver. I first used gobuster to enumerate possible and interesting folders to exploit.

<figure><img src="../../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

The `db.php` file was the most interesting. I also took a look at the `/resources` endpoint to find some cool stuff. The README file has some hints towards finding a tracker of some sort.

<figure><img src="../../../.gitbook/assets/image (167).png" alt=""><figcaption></figcaption></figure>

Then, the `bountylog.js` file revealed an interesting endpoint.

<figure><img src="../../../.gitbook/assets/image (205).png" alt=""><figcaption></figcaption></figure>

It seems that POST requests are being sent to the `tracker.php` endpoint and processed. So this means our XXE payloads have to be sent there.

Then, using `php://filter/convert.base64-encode/resource=db.php`, we can extract the file.

<figure><img src="../../../.gitbook/assets/image (195).png" alt=""><figcaption></figcaption></figure>

After decoding the file, we find a password.

<figure><img src="../../../.gitbook/assets/image (191).png" alt=""><figcaption></figcaption></figure>

We can then read the `/etc/passwd` file to see which users have a `/bin/bash` shell on the machine.

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

It seems that the `development` user was the target here. We can combine this and the credentials found to SSH into the machine.

<figure><img src="../../../.gitbook/assets/image (194).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

Initial enumeration for `sudo` privileges revealed this:

<figure><img src="../../../.gitbook/assets/image (193).png" alt=""><figcaption></figcaption></figure>

We can read this file to that it has a vulnerbale function being used.

<figure><img src="../../../.gitbook/assets/image (179).png" alt=""><figcaption></figcaption></figure>

### Eval Exploit

Since this was using `eval`, it means that we can include some code here and the script would execute it. First, we would need to create a valid 'ticket' as per the machine's instructions. Based on the code, this right here works:

<figure><img src="../../../.gitbook/assets/image (206).png" alt=""><figcaption></figcaption></figure>

To achieve RCE, we would first need to include a `true` condition to cause the `exec` function to short-circuit and move on to the next bit.

What I did was use the `and` operator to incude a short Python script to spawn a root TTY shell.

<figure><img src="../../../.gitbook/assets/image (197).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>
