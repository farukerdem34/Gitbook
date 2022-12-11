---
description: Taken from UHC.
---

# Backend (WIP)

## Gaining Access

We start with an Nmap scan as usual:

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

From here, we can view Port 80.&#x20;

### Port 80:

Here, we can view an API that we need to exploit.

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Starting with a Gobuster, we can find out the different endpoints for this:

&#x20;

<figure><img src="../../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

At the `/api/v1` endpoint, we find that that there is a `/user` endpoint that we can fuzz further using `wfuzz`.

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Writing in Progress.

