# Windows Authentication

Windows authentication is not as straightforward, and it's made a little more complex thanks to Active Directory and having multiple machines use the same set of credentials. Understanding how Windows authenticates users is critical to allowing us to move laterally in a domain.&#x20;

## Types of Authentication

There are different methods of which a user can authenticate to a Windows machine that may or may not be connected to a domain.

### Local&#x20;

Local users are only present in one **specific standalone system**. Obviously, my user isn't present in my friend's laptop, and it is only present in my laptop. Likewise, `laptop1\user` and `laptop2\user` are totally different users.&#x20;

The local users have their records and passwords stored in the **Security Account Manager (SAM) database**. Windows would use these records to verify passwords when trying to authenticate to a system. This is similar to how `/etc/passwd` works in checking users:

1. User attempts to login with `user:Password@123`.
2. Laptop checks SAM for whether `user` exists and if `Password@123` is valid.
3. If yes, grant access. If no, deny.&#x20;

### Domain&#x20;

Domain users are only present in **one specific domain**. For example, `acme\user123` is not present in `acme2\user123`. However, **all domain-joined devices** know how to handle authentication. This means that, by default, **any domain user can login to any domain computer** provided they have physical access to it.&#x20;

Domain joined devices delegate the authentication to the DC, and all of these records are stored within the NT Directory Service (NTDS) database. The DC uses the NTDS records to authenticate users.&#x20;

1. User attempts to login with `acme\user:Password@123` on `WKSTN`.&#x20;
2. `WKSTN` delegates the authentication to the `DC`.
3. `DC` checks NTDS to verify that the user exists and password is correct,
4. `DC` verifies user and sends a thumbs up to `WKSTN`.&#x20;
5. `WKSTN` grants access.

### Remote&#x20;

Remote authentications require privileges to do (for example, being part of the Remote Management Group allows users to have remote Powershell sessions). When moving laterally, **this is the main thing we care about**.&#x20;

## Windows Authentication

### Terms

Some terms we need to know before discussing how the authentication works:

#### Auth Packages

Authentication Packages (APs) authenticate users by analysing their **logon data via Security Support Providers (SSP)**. Different APs provide support for different logon processes, and they are typically `.dll` files that are loaded and used by **Local Security Authority (LSA)**.&#x20;

Some examples of APs are:

* Kerberos
* NTLM Hashes

When we try to login to a website, the website is probably using some kind of backend database that is verifying our credentials. APs provide the logic needed for Windows to act as **both of these** using the same machine.&#x20;

#### LSA

As mentioned earlier, LSA allows Windows to act as both the client and authentication server at the same time.

How it works is illustrated in this diagram here (taken from ATTL4S):

<figure><img src="../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

#### SSP

Microsoft provides an Interface for the SSPs (SSPI) to integrate applications with this authentication system.&#x20;

<figure><img src="../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

There are 4 major categories for the functions provided:

1. Package Management --> Handles packages
2. Credential Management --> Handles credentials of principals
3. Context Management --> Creates security context
4. Message Support --> Ensures message **integrity and privacy** during message exchanges.

### Authentication

When someone successfully authenticates to the WIndows message, the AP does 2 things:

1. Create a new logon session
2. Provides security information about the authenticated user to the LSA

The LSA then creates an access token **representing the user's local security context** on the system. This security context would contain information like:

* User SID
* Logon session SID
* Integrity
* Groups
* Other user information

This security context is used to create an access token, which is the thing required to create the new logon session.&#x20;

### Sessions

There are 2 main types of sessions that are created:

1. Interactive --> Credentials given
2. Non-interactive --> Credentials not given

#### Interactive

Interactive sessions are what happens when we login normally through our login page. The user credentials are cached within the memory of the LSA process, called the Local Security Authority Subsystem Service (LSASS). In specific, it is cached in `lsass.exe`. Cached credentials allow for Windows to provide for a Single Sign-On (SSO) service to users.&#x20;

<figure><img src="../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

#### Non-Interactive

These are sessions that leverage the cached credentials on behalf of a user. When we use the SSO service to logon to the same or another device, this is actually a non-interactive session. **Interactive sessions need to happen first for credentials to be cached**.&#x20;

This leverages the use of the SSPI to work.

<figure><img src="../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

For example, a non-interactive session can grant us access to the file system of another device and do `ls \\dc\c$` using cached credentials.&#x20;

After a user successfully authenticates, a new **logon session** is created regardless of what type of authentication is used. The cached credentials in the AP are tied to logon sessions. Logon sessions are not limited to the 2 types listed above:

<figure><img src="../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

We can verify a logon session with `Get-LogonSession` from Powerview.

### Access Tokens

The information that is returned to LSA after creating the logon session is used to create an access token. An access token is a protected object that contains the **local security context** of an authenticated user. The security context is defined as the **privileges and permissions a user has on a specific workstation and across the network.**&#x20;

<figure><img src="../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

Every single logon session is identifiable by a 64-bit locally unique identifier (LUID), otherwise known as the logon ID. This access token contains an Authentication ID (AuthID) that identifies the logon session via the LUID.

<figure><img src="../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

The access token caches a number of attributes that determine its security context such as the user SID, group memberships, privileges and logon ID that references the origin logon session. In the above image, we can see that the Integrity is set to Medium. What if use the 'Run as Administrator' option and run `cmd.exe`?&#x20;

This is where multiple security contexts come in. One user can have more than 1 context, and using the 'Run as Administrator' option simply spawns a **high integrity** process via the User Account Control (UAC). As such, Windows allows users to have different access tokens and logon sessions **in the same system**.&#x20;

These access tokens serve as an access check for Windows. For instance, when a thread or process attempts to read a high security object (like `root.txt`), it needs 3 pieces of information:

1. Who is requesting access?
2. What do they want to do (read, write or execute)?
3. Who can access this object?

Windows will first check the token associated with the calling thread and look at the authorization attributes cached. Then, it looks at the access requested by the thread. Lastly, Windows retrieves the security descriptor for the target object in the form of a DACL specifying what users have access to the object and what type of access is granted.&#x20;

If the access token matches what is required, then access is granted, else it is not.&#x20;
