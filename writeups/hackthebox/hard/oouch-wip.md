---
description: OAuth 2.0 is complex and cool
---

# Oouch (WIP)

## Gaining Access

Nmap scan:

```
$ nmap -p- --min-rate 5000 10.129.29.195
Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-09 00:16 EST
Nmap scan report for 10.129.29.195
Host is up (0.022s latency).
Not shown: 65531 closed tcp ports (conn-refused)
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
5000/tcp open  upnp
8000/tcp open  http-alt
```

### FTP Anonymous Accss

I checked for anonymous access to the FTP server, and it works. We can download a `project.txt` file from it. These are the contents:

```
$ cat project.txt 
Flask -> Consumer
Django -> Authorization Server
```

Might need this information later. Also, we can see that this is the user `qtc` server.

<figure><img src="../../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

### Port 5000

This webpage just shows a login page:

<figure><img src="../../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

I registered a user and logged in to see the dashboard.

<figure><img src="../../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

There are 3 main functions, a Password Change, Documents and the Contact one. The Password Change is not interesting, Documents are only available for the administrator user. That just leaves  the Contact function.

<figure><img src="../../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

This looks like an XSS platform to somehow steal the administrator cookie. When trying to submit a basic XSS payload, this is what I got:

<figure><img src="../../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

This might not be the way. Instead, I ran a `gobuster` scan on the machine to see what other endpoints are hidden. I found a weird `/oauth` endpoint that could be of use: (this took a lot of wordlists to do)

```
$ gobuster dir -w /usr/share/seclists/Discovery/Web-Content/big.txt  -u http://10.129.29.195:5000/ -t 150 | grep -v 502 
===============================================================
Gobuster v3.3
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.29.195:5000/
[+] Method:                  GET
[+] Threads:                 150
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.3
[+] Timeout:                 10s
===============================================================
2023/03/09 00:37:10 Starting gobuster in directory enumeration mode
===============================================================
/about                (Status: 302) [Size: 247] [--> http://10.129.29.195:5000/login?next=%2Fabout]
/contact              (Status: 302) [Size: 251] [--> http://10.129.29.195:5000/login?next=%2Fcontact]
/documents            (Status: 302) [Size: 255] [--> http://10.129.29.195:5000/login?next=%2Fdocuments]
/home                 (Status: 302) [Size: 245] [--> http://10.129.29.195:5000/login?next=%2Fhome]
/logout               (Status: 302) [Size: 219] [--> http://10.129.29.195:5000/login]
/login                (Status: 200) [Size: 1828]
/oauth                (Status: 302) [Size: 247] [--> http://10.129.29.195:5000/login?next=%2Foauth]
/register             (Status: 200) [Size: 2109]
/profile              (Status: 302) [Size: 251] [--> http://10.129.29.195:5000/login?next=%2Fprofile]
```

We can head to that and see what it does.

### Oauth

This gives us instructions on how to connet to the OAuth server running on this machine:

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

When trying to access it, it attempts to access `authorization.oouch.htb:8000`. So the OAuth server is running on port 8000.

Using the connect options shows us this:

<figure><img src="../../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

When we click authorize, we just are logged in as the same user it seems.&#x20;

### Authorisation Server&#x20;

On Port 8000, all we see is this:

<figure><img src="../../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

When are visiting this port from port 5000 via OAuth, this is the request that gets sent:

```http
GET /oauth/authorize/?client_id=UDBtC8HhZI18nJ53kJVJpXp4IIffRhKEXZ0fSd82&response_type=code&redirect_uri=http://consumer.oouch.htb:5000/oauth/connect/token&scope=read HTTP/1.1
Host: authorization.oouch.htb:8000
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://oouch.htb:5000/
Connection: close
Cookie: csrftoken=CHNQoemrMXB0fSL1ww9wI1NPq7GSYoCHqWWxW9FW7ZQpfflFeHATfHu00krDHJ8i
Upgrade-Insecure-Requests: 1

```

And this is the page that gets shown to us:

<figure><img src="../../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

When visiting `http://authorization.oouch.htb:8000`, we can see how to register to this server:

<figure><img src="../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

In case you're unaware of OAuth 2.0, you can read this:

{% embed url="https://portswigger.net/web-security/oauth" %}

It basically is the service that allows us to 'Login with Facebook' or other social media. The exploit here is to somehow use this faulty OAuth implementation to login as the administrator of the website and view the documents, which might contain some useful information.

For now, we can register a user and keep enumerating this website. This is the page we get to after logging in:

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

Clicking these two options don't seem to do much. I ran a `gobuster` scan once again.

```
$ gobuster dir -w /usr/share/seclists/Discovery/Web-Content/big.txt  -u http://authorization.oouch.htb:8000/oauth -t 150 | grep -v 502
===============================================================
Gobuster v3.3
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://authorization.oouch.htb:8000/oauth
[+] Method:                  GET
[+] Threads:                 150
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/big.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.3
[+] Timeout:                 10s
===============================================================
2023/03/09 00:47:42 Starting gobuster in directory enumeration mode
===============================================================
/applications         (Status: 301) [Size: 0] [--> /oauth/applications/]
/authorize            (Status: 301) [Size: 0] [--> /oauth/authorize/]
```

We can see a new endpont at `/applications`. Trying to access this requires credentials:

<figure><img src="../../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

I did not have any credentials for now. After creating this account and re-testing, using the `/oauth/connect` function earlier now shows a different user profile.

<figure><img src="../../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

The website recognises my new OAuth account that I created and is considered 'connected'.&#x20;

### OAuth Forced Linking

Here's an overview of how exactly OAuth works:

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

We can view the HTTP requests throguh Burpsuite to see what exactly is happening.

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

These 4 requests are essentially the OAuth mechanism, and there is the access token in a POST request. Our goal is to authenticate as `qtc`, since he probably is the administrator user of this machine. As such, we can use a CSRF attack because this implementation of OAuth does not seem to send the `state` parameter.&#x20;

This attack would force the website to link my account with the `qtc` account, giving me access over his resources. Here's a Portswigger Lab that has the same attack method:

{% embed url="https://portswigger.net/web-security/oauth/lab-oauth-forced-oauth-profile-linking" %}

WIP!&#x20;
