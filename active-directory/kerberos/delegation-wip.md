# Delegation (WIP)

Personally, Kerberos and its delegation mechanisms have always been a little hazy for me, and I wanted to take the time to understand it. Basically, this page is my attempt of understanding Delegation, why its used and attacks on it phrased in my own words.&#x20;

## Double Hop Problem

Suppose a user logs in to a machine, let's say the `WEB` machine, but how does the `WEB` machine authenticate to the **same** server and **access the database folders?** This is where's Windows Authentication mechanism comes in.

This idea of "acting on the behalf of another user" is possible thanks to Windows's **Client Impersonation** feature as part of authentication. When we attempt to connect to the web app, credentials are verified, and **an acces token with the security context of the user is created**.

When we enter the credentials of the user, this would be compared against the credentials stored in `lsass.exe` and create a **new user logon session and access token on the target system**. The service places a **copy of that Token into a new thread, and this new thread can act on the user's behalf**. Since we have credentials, the new logon session would have credentials and thus we can operate normally.&#x20;

This token thing is also the reason why we can 'steal tokens' from processes running as other users to impersonate them and run commands as them.&#x20;

**However, this assumes that the resources we want to access are on the SAME SERVER.** For example, if the `C:\Database` and `C:\Web` folders are both located within the `WEB` machine, then  just having the user's password is enough.&#x20;

If there are 2 different servers, say `WEB` and `DB`, then this method of authentication would not work. `lsass.exe` would not normally store the credentials of another user from another machine. A new logon session is thus still created, **but because we have no credentials, we cannot go anywhere with this session**. This is known as the **Double Hop Problem**.&#x20;

## Delegation&#x20;

Delegation basically allows a user or machine to act on the behalf of another user to another service. One common implementation is where a user authenticates to a front-end web application that serves a back-end database.

The front-end needs to authenticate to the back-end database (using Kerberos) as the authenticated user. This is how delegation works in a nutshell:

<figure><img src="../../.gitbook/assets/image (714).png" alt=""><figcaption></figcaption></figure>

### Password / NTLM Authentication?

Delegation allows for the impersonation of users **in the network and not only locally**. Access tokens that are created using credentials are for local purposes only. For this, we can use **credential delegation,** and one common example is Remote Powershell Sessions.&#x20;

This method involves sending some kind of credential material to the service, so that the service can impersonate clients in the network. Here are some commands that make use of this feature:

```powershell
#Predefine necessary information
$Username = "YOURDOMAIN\username"
$Password = "password"
$ComputerName = "server"
$Script = {notepad.exe}
#Create credential object
$SecurePassWord = ConvertTo-SecureString -AsPlainText $Password -Force
$Cred = New-Object -TypeName "System.Management.Automation.PSCredential" -ArgumentList $Username, $SecurePassWord
#Create session object with this
$Session = New-PSSession -ComputerName $ComputerName -credential $Cred
# Enter-PSSession
$job = invoke-command -session $session -scriptblock $script
echo $job
```

This solves the Double Hop problem, because we can access another user and run commands as a user on the network instead of just locally. This method would also require the user to be part of the **Remote Management Group** on the network.&#x20;

Obviously, this is not the most secure method as sending credentials in plaintext is never a good idea, especially for service accounts (which generally have high permissions over certain resources in the domain). Sending NTLM hashes is also not great because Pass The Hash attacks exist. As such, it is normal for NTLM authentication to be disabled completely.

{% embed url="https://blog.quest.com/ntlm-authentication-what-it-is-and-why-you-should-avoid-using-it/" %}

In the HTB machine Hathor, NTLM authentication (and hence passwords) are completely disabled, leaving us with only Kerberos authentication. Kerberos authentication, while still risky, **does not depend on the user's original password or NTLM hashes**. The authentication is based on tickets and session keys, and they are trusted by default.&#x20;

Having tickets and session keys cached on a server is relatively more secure compared to passwords or hashes in general. This is why in some systems, only Kerberos authentication is allowed.&#x20;

There are 3 main types of delegation available in AD systems:

1. Unconstrained Delegation
2. Constrained Delgation
3. Role-Based Constrained Delegation (RBCD)&#x20;

Exploitation of any of these privileges is not a CVE, but rather an **abuse of a service**. This uses the intentional features of the system against itself.&#x20;

## Unconstrained Delegation

This is the first type of delegation introduced in Windows 2000. When configured, the KDC would include a copy of the user's TGT **inside the TGS**. when the user accesses the `DB` machine, it extracts the user's TGT from the TGS and caches it in memory. Then, it would use this TGT to request for a TGS, which would allow for the accessing of database resources.

<figure><img src="../../.gitbook/assets/image (713).png" alt=""><figcaption></figcaption></figure>

The service can act on behalf of the client in the network **simply by using its TGT**. This feature requires the `SeEnableDelegation` privilege to be enabled. We can configure this like so:

<figure><img src="../../.gitbook/assets/image (712).png" alt=""><figcaption></figcaption></figure>

