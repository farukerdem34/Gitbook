# Resourced (PG)

Resourced (PG)

Friday, 1 April 2022

4:33 pm

Interesting box,

&#x20;

Enum4linux reveals a bunch of users and one particular password that we could use to check the SMB shares. One SMB share was called Password Auditing.

&#x20;

From there, we are able to get out the hashes for multiple users.

&#x20;

We can then do a ldapdomaindump and see which users are within the Remote Mangement Group.

&#x20;

We identify the user as L.Livingstone and then evil-winrm in as the user.

&#x20;

From there, the user has GenericAll permissions on the entire domain. What this means is, essentially, the user can create new accounts that can be created to allow for new accounts to impersonate and authenticate as any domain user on the directory.

&#x20;

In human terms, the device would basically grant access to all accounts, including administrators to all accounts present on the directory. The domain would trust the user to account and hence we can request Kerberos tickets that would allow us to login.

&#x20;

We can follow the guide [here](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/resource-based-constrained-delegation-ad-computer-object-take-over-and-privilged-code-execution) and then be able to extract the administrator.ccache.

&#x20;

I activated the walkthrough and saw that this is indeed the way, but they did so differently and also with a better method.

&#x20;

!\[\[17\_Resourced (PG)\_image001.png]]

With their account, they were able to extract the tickets and stuff easily for us to log in with.

&#x20;

This would grant us the administrator access to the AD account.

&#x20;

The methodology I used to gain user access was good, however I had to activate a hint to progress as I did not use Bloodhound correctly and saw the wrong information.

&#x20;

This did not allow me to find the misconfigured permission and hence I was unable to properly find the PE vector.

&#x20;
