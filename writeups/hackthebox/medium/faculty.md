# Faculty

## Gaining Access

As usual, we start with a Nmap scan.&#x20;

<figure><img src="../../../.gitbook/assets/image (2) (2) (3).png" alt=""><figcaption></figcaption></figure>

There's the `faculty.htb` domain running on port 80. We can add this to the `/etc/hosts` file.&#x20;

### SQL Injection

Basic PHP login page for a Faculty Scheduling System is present here:

<figure><img src="../../../.gitbook/assets/image (9) (1) (3) (1).png" alt=""><figcaption></figcaption></figure>

Checking for common directories such as the `/admin` endpoint reveals another login page.

<figure><img src="../../../.gitbook/assets/image (6) (1) (3) (2).png" alt=""><figcaption></figcaption></figure>

Proxying the traffic in Burp, sending a `'` character as a username triggers an SQL error.

<figure><img src="../../../.gitbook/assets/image (5) (1) (3) (1).png" alt=""><figcaption></figcaption></figure>

Now that we have confirmed SQL Injection is present, we can dump out all the tables within the database from this using `sqlmap`.

<figure><img src="../../../.gitbook/assets/image (1) (2) (3) (1).png" alt=""><figcaption></figcaption></figure>

The `users` table had an `Administrator` user with a hashed password, however the hash cannot be cracked.

<figure><img src="../../../.gitbook/assets/image (3) (1) (4) (1).png" alt=""><figcaption></figcaption></figure>

In this case, we can dump the `faculty` table and attempt to login via the original login method. There's a PIN number associated with the `Administrator` user and we can use that to login.

<figure><img src="../../../.gitbook/assets/image (176).png" alt=""><figcaption></figcaption></figure>

### PDF Generator LFI

WItin the website, there's a PDF Generator that would display certain courses.&#x20;

<figure><img src="../../../.gitbook/assets/image (7) (2) (4).png" alt=""><figcaption></figcaption></figure>

We can download this PDF and use `exiftool` to find out more information about it.

<figure><img src="../../../.gitbook/assets/image (74) (2) (1).png" alt=""><figcaption></figcaption></figure>

mPDF 6.0 is vulnerable to an LFI exploit that would allow for us to read the files on the server. With this, we can have to send this payload encoded with Base64:

{% code overflow="wrap" %}
```
<annotation file="/etc/passwd" content="/etc/passwd" icon="Graph" title="Attached File: /etc/passwd" pos-x="195" />
```
{% endcode %}

This would load a PDF file that contains the `/etc/passwd` file.

<figure><img src="../../../.gitbook/assets/image (182).png" alt=""><figcaption></figcaption></figure>

The `/etc/passwd` file would be attached to the main PDF like this:

<figure><img src="../../../.gitbook/assets/image (356) (1).png" alt=""><figcaption></figcaption></figure>

We can then read it to find out the users that are present on the machine.

<figure><img src="../../../.gitbook/assets/image (358) (1).png" alt=""><figcaption></figcaption></figure>

From here, we can try to find the `admin.php` or `db.php` file within the machine to find some credentials. Doing a quick `gobuster` scan reveals that there is a `db_connect.php` file that we can read to find a password.

<figure><img src="../../../.gitbook/assets/image (360) (1).png" alt=""><figcaption></figcaption></figure>

With this, we can `ssh` in as the `gbyolo` user using the password found.&#x20;

## Privilege Escalation

### Meta-Git RCE

The other user found on the machine is the `developer` user. When checking the `sudo` privileges of the `gbyolo` user, we find this:

<figure><img src="../../../.gitbook/assets/image (353) (1).png" alt=""><figcaption></figcaption></figure>

`meta-git` is a binary that allows us to clone repos as per git. However, this was vulnerable to an RCE exploit, meaning we can execute commands as `developer`.&#x20;

{% embed url="https://hackerone.com/reports/728040" %}

Using the PoC, we can read the `id_rsa` file from the user's home directory.

<figure><img src="../../../.gitbook/assets/image (351) (1).png" alt=""><figcaption></figcaption></figure>

Then, we can SSH in as `developer`.&#x20;

### GDB Attaching

Doing initial enumeration reveals that the `developer` user is part of the `debug` group.

<figure><img src="../../../.gitbook/assets/image (355) (1).png" alt=""><figcaption></figcaption></figure>

Running LinPEAS also reveals that we can run GDB as we are part of the `debug` group.

<figure><img src="../../../.gitbook/assets/image (352) (1).png" alt=""><figcaption></figcaption></figure>

In this case, `gdb` can be used to attach ourselves to a process **run by root** and spawn more child processes. These child processes would all be running as root and we can basically gain RCE as root in this manner. This would stop the execution process and make the root thread spawn whatever other functions that we want.&#x20;

First, we would need to find a process running as root, and preferably a Python or Bash one so we can execute commands easily.

<figure><img src="../../../.gitbook/assets/image (357) (1).png" alt=""><figcaption></figcaption></figure>

The PID in this case is `621`, so we can run `gdb -p 621` to attach ourselves there. Afterwards, I called the `system()` function to spawn a reverse shell as root.&#x20;

<figure><img src="../../../.gitbook/assets/image (354) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (359) (1).png" alt=""><figcaption></figcaption></figure>
