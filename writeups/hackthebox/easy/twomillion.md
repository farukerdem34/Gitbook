---
description: Happy 2 Million HTB!
---

# TwoMillion

## Gaining Access

Nmap scan:

```
$ nmap -p- --min-rate 5000 10.129.76.193
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-15 18:45 +08
Nmap scan report for 10.129.76.193
Host is up (0.011s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

We have to add `2million.htb` to our `/etc/hosts` file to view the web application.

### HTB Replica

The website resembles the actual live HTB platform:

<figure><img src="../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

The website also shows how long has HTB come since the start:

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

Anyways, we can attempt to register a user on this site and maybe find some sort of access control weakness. On the main page, there is a 'Join HTB' button, but it requires an invite code to access:

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

I didn't have an invite code, so we'll have to leave this for now. I also don't have any credentials to register a user, so the website's applications have limited use as of now. We can do a directory and subdomain scan for this site. I ran a `feroxbuster` directory scan and a `wfuzz` subdomain scan. The `feroxbuster` scan returned some interesting stuff:

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

There was a `register` directory present.&#x20;

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

I couldn't register an account because I still didn't have an invite code at all. I looked at the account in Burpsuite, and found this at the bottom of the page:

<figure><img src="../../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

There's an `inviteapi.min.js` file that looks custom. Here's the contents of that file:

{% code overflow="wrap" %}
```javascript
eval(function(p,a,c,k,e,d){e=function(c){return c.toString(36)};if(!''.replace(/^/,String)){while(c--){d[c.toString(a)]=k[c]||c.toString(a)}k=[function(e){return d[e]}];e=function(){return'\\w+'};c=1};while(c--){if(k[c]){p=p.replace(new RegExp('\\b'+e(c)+'\\b','g'),k[c])}}return p}('1 i(4){h 8={"4":4};$.9({a:"7",5:"6",g:8,b:\'/d/e/n\',c:1(0){3.2(0)},f:1(0){3.2(0)}})}1 j(){$.9({a:"7",5:"6",b:\'/d/e/k/l/m\',c:1(0){3.2(0)},f:1(0){3.2(0)}})}',24,24,'response|function|log|console|code|dataType|json|POST|formData|ajax|type|url|success|api/v1|invite|error|data|var|verifyInviteCode|makeInviteCode|how|to|generate|verify'.split('|'),0,{}))
```
{% endcode %}

This looks like it generates some kind of token. Within it, we can see that it uses a `makeInviteCode` function. This file is loaded at the `/invite` directory, which is where we need to submit a code. Within the Javascript Console in Inspector tools, I ran that function.

<figure><img src="../../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

Interesting!&#x20;
