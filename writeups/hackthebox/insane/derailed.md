# Derailed

## Gaining Access

As usual, we start with an Nmap scan:

<figure><img src="../../../.gitbook/assets/image (98).png" alt=""><figcaption></figcaption></figure>

Port 3000 was found to be a HTTP port leading us to this Clipnotes page.

### Directory Enum

<figure><img src="../../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

Wasn't much to play around with, as we had no credentials yet. Decided to run a directory scan to find if there are any endpoints. Eventually, I found this /rails endpoint, using dirsearch.

<figure><img src="../../../.gitbook/assets/image (99).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

Interesting. This presented a lot of information for me and also tells me this is a Ruby on Rails project.

<figure><img src="../../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

From here, we can try to fuzz out other information and endpoints on this /rail directory. I used feroxbuster for its recursive search function.

<figure><img src="../../../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

This directory basically shows us every single path there was in the website:

<figure><img src="../../../.gitbook/assets/image (101).png" alt=""><figcaption></figcaption></figure>

### /clipnotes

Earlier, we saw some form of clipnote function. Testing it shows us that each time we create a new one, it is stored on the server somewhere. Notice that this one I created was 110.

<figure><img src="../../../.gitbook/assets/image (102).png" alt=""><figcaption></figcaption></figure>

&#x20;Using the **/clipnotes/raw/:id** format, I was able to view the first clipnote, submitted by a user called Alice. Visiting anything other than 1 was not possible.

<figure><img src="../../../.gitbook/assets/image (100).png" alt=""><figcaption></figcaption></figure>

I was interested in what other number is present, so I used wfuzz to enumerate out all other numbers. None are present it seems

<figure><img src="../../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

I checked out the other endpoints, as there may be more interesting ones. The **/report** one looks good.

### /report

<figure><img src="../../../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

Submitting a report reveals tells us that an admin would look at it. This tells me that perhaps, there is a form of XSS present on the site.

When looking at the POST request for the report, we can see that it sends this authenticity\_token:

<figure><img src="../../../.gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

However, the cookies have been set to HttpOnly, meaning that stealing cookies is pointless in this case. XSS on the administrator could either allow us to enumerate more about the /administration page, or simply to steal his cookie and impersonate him.

Because this is clearly an XSS challenge, I thought of first finding a potential XSS entry.&#x20;

### Finding XSS Point

I messed around a lot with the clipnotes and tried all sorts of stuff, but it wasn't loading Javascript. Then I realised that the **author** of the clipnotes was something that I controlled. Perhaps, I could overflow the thing or try to register a malicious user. Since the page renders that username, this could potentially be vulnerable.

So I started a HTTP server, and attempted this:

<figure><img src="../../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

The reason I did this was because I am aware that there is a potential limit to the username, and trying to overflow that may cause the end bit to be rendered as JS code. Then I created a clipnote:

<figure><img src="../../../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

The overflow kind of worked, managed to remove the last portion about the created bit. I then went to try various different payloads including \<img> tags and stuff. DIdn't really work. I then suspected this has to do with some kind of CVE that was released recently (usual pattern of HTB, uses CVEs from 2022), and went hunting for Ruby + XSS related exploits that came out recently.

I eventually found CVE-2022-32209, which was a XSS exploit for Rails::Html::Sanitizer. &#x20;

{% embed url="https://groups.google.com/g/rubyonrails-security/c/ce9PhUANQ6s?pli=1" %}

Seems that the \<select> tag was being used maliciously. I then tried the same overflow trick with this payload:

```
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa<select<style/><img src='http://10.10.14.29/xss.pls'>
```

<figure><img src="../../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

This worked! I was able to get a callback as well:

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Now, we just need to find a way to exploit this XSS.

### CSRF + XSS

Understanding that we now have XSS, I was thinking of using CSRF in order to retrieve more information about the /administration page. The reason CSRF is used is because **CSRF tokens do not protect against XSS.**&#x20;

Since we can basically execute basic web requests using our username, I thought of using the clipnotes itself to house my Javascript code. This was to be a basic exploit, just to view the /administration page and return it to my listening port.

Work in Progress!
