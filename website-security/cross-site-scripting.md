# Cross-Site Scripting

Cross-Site Scripting, or XSS, is a vulnerability that would allow attackers to exploit the interactions that **other users** have with an application. In short, it can allow us to impersonate and perform actions as another user, steal information and in some cases, gain RCE.

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

## How it Works

Browsers and websites allow users to interact with them and perform actions through Javascript. XSS would work via injecting malicious Javascript code to execute on a user's browser. Basically, **scripting across the website**.

There are 3 types of XSS:

1. Reflected XSS
2. Stored XSS
3. DOM Based XSS

### Reflected XSS

Reflected XSS is the simplest form of the exploit. This occurs when a malicious script is reflected off a web application and onto the victim's browser. This script is normally activated through a link or action on the website and would be redirected to the next user.

<figure><img src="../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

Now, suppose that we have a website that would display some forms of&#x20;
