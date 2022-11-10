# Return (HTB)

Return (HTB)

Saturday, 26 March 2022

2:01 pm

AD machine that had a page that would connect to other IP addresses.

&#x20;

From there, set up Responder and intercept the response, which provided for the password to evil-winrm in as the user.

&#x20;

Once we are in, there are services which we can change the binary path to, allowing for the system to execute a reverse shell as the administrator.

&#x20;

For the CTF side, we did have the backup and restore privileges, meaning we could just robocopy /b the root.txt and SAM out, which was an alternative solution. Technically, CTF-wise it's doable without the administrator account.

&#x20;

&#x20;
