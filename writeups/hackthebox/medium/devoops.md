# Devoops

## Gaining Access

Nmap scan:

<figure><img src="../../../.gitbook/assets/image (14) (1) (2) (3).png" alt=""><figcaption></figcaption></figure>

There's only one port open on this machine.

### XXE Injection

On port 5000, this is what we see:

<figure><img src="../../../.gitbook/assets/image (13) (4).png" alt=""><figcaption></figcaption></figure>

We can use `gobuster` on this to find more directories.

<figure><img src="../../../.gitbook/assets/image (19) (4).png" alt=""><figcaption></figcaption></figure>

`/feed` would bring us here:

<figure><img src="../../../.gitbook/assets/image (31) (6) (2).png" alt=""><figcaption></figcaption></figure>

XML Injection is pretty helpful, and i noticed that when we upload a file using this API, a POST request would be sent to the `/upload` directory with HTTP form data.

However, trying to send any XML files that I created results in a Internal Server Error message being returned. Turns out, there are specific elements that we need to use for this endpoint:

<figure><img src="../../../.gitbook/assets/image (27) (6) (1).png" alt=""><figcaption></figcaption></figure>

With these, we can wrap them in another tag and start getting successful uploads through.

<figure><img src="../../../.gitbook/assets/image (39) (7).png" alt=""><figcaption></figcaption></figure>

From this, we identified that we have a user called `roosa`. Then, we can attempt some basic XXE LFI payloads to read the user's private SSH key.

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (2).png" alt=""><figcaption></figcaption></figure>

Afterwards, we can simply SSH into the machine.

<figure><img src="../../../.gitbook/assets/image (26) (6).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

### Git Logs

When checking the `id` of this user, we see that we are part of the `sudo` group.

<figure><img src="../../../.gitbook/assets/image (255) (2).png" alt=""><figcaption></figcaption></figure>

Within the home directory of the user, we also find some Git repository files.

<figure><img src="../../../.gitbook/assets/image (9) (1) (2).png" alt=""><figcaption></figcaption></figure>

Using `find /home -name .git`, we can find the specific location of the Git repository to read its logs.

<figure><img src="../../../.gitbook/assets/image (2) (1) (3) (3).png" alt=""><figcaption></figcaption></figure>

After heading to that directory, we would find an SSH key after using `git log -p -2`:

<figure><img src="../../../.gitbook/assets/image (256) (2).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (17) (5) (2).png" alt=""><figcaption></figcaption></figure>

Surprisingly, this was the root SSH key.

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (2).png" alt=""><figcaption></figcaption></figure>
