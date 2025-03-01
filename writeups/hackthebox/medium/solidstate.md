# SolidState

## Gaining Access

Nmap scan:

<figure><img src="../../../.gitbook/assets/image (34) (1) (4).png" alt=""><figcaption></figcaption></figure>

### JAMES Server

SMTP is open, which is rather suspicious. I connected via `nc` and tested some default credentials, and found that `root:root` worked.

<figure><img src="../../../.gitbook/assets/image (3) (1) (2) (3).png" alt=""><figcaption></figcaption></figure>

Now that we are logged in, we can read some emails:

<figure><img src="../../../.gitbook/assets/image (20) (2) (1).png" alt=""><figcaption></figcaption></figure>

With this, we can SSH in as `mindy`.

### Shell Escape

When in the user's directory, we find a restricted shell where we cannot execute a lot:

<figure><img src="../../../.gitbook/assets/image (28) (1) (5).png" alt=""><figcaption></figcaption></figure>

I researched a bit on how to escape this shell, and found that appending `-t "bash --noprofile"` works:

<figure><img src="../../../.gitbook/assets/image (8) (1) (3).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

### Cronjob Injection

We can run `pspy32` on this machine to view processes:

<figure><img src="../../../.gitbook/assets/image (36) (2) (2).png" alt=""><figcaption></figcaption></figure>

I found that we have write access over this file, so we can just append a reverse shell to it:

<figure><img src="../../../.gitbook/assets/image (22) (1) (3).png" alt=""><figcaption></figcaption></figure>

After waiting for a bit, we would catch a reverse shell:

<figure><img src="../../../.gitbook/assets/image (18) (3) (2).png" alt=""><figcaption></figcaption></figure>
