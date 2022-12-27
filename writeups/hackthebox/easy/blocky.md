# Blocky

## Gaining Access

Nmap scan:

<figure><img src="../../../.gitbook/assets/image (25) (5) (1).png" alt=""><figcaption></figcaption></figure>

### Plugins

Port 80 is a Wordpress Site that has a post referencing a plugin and a wiki system being in development.

<figure><img src="../../../.gitbook/assets/image (3) (1) (6).png" alt=""><figcaption></figcaption></figure>

&#x20;We can use `gobuster` on the website to find some hidden content.

<figure><img src="../../../.gitbook/assets/image (7) (2).png" alt=""><figcaption></figcaption></figure>

Heading to the plugins directory, we find two .jar files.

<figure><img src="../../../.gitbook/assets/image (19) (1) (3).png" alt=""><figcaption></figcaption></figure>

We can take a look at these jar files using `jd-gui`, and find some SQL credentials within the machine.

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

So now we have a password but no user to use it with.

### Wordpress Scan

Earlier, we found some Wordpress-related directories, hence we can use `wpscan` to enumerate more about this machine. This would allow us to find this `notch` user.

<figure><img src="../../../.gitbook/assets/image (21) (6).png" alt=""><figcaption></figcaption></figure>

With the password and this username, we can SSH into the machine.

<figure><img src="../../../.gitbook/assets/image (18) (1) (1).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

Checking sudo privileges, we see this.

<figure><img src="../../../.gitbook/assets/image (23) (6).png" alt=""><figcaption></figcaption></figure>

Because we have the password from earlier, we can run `sudo su` to become root.
