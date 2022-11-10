# BlackField (HTB)

1. First would be using null credentials to find some users.
2. After a user list is determined, we can use that to ASREP Roast and find a hash for the 'support' user.
3. From there, we can use bloodhound to find more information, and determine that the support user is able to change the password of the audit2020 user.
4. Once the password is changed via RPCClient, we can look into the audit2020 user and find out more about the shares that can be accessed. We are able to grab one lsass.zip and get out the lsass.dmp file.
5. This can be transferred to a Windows machine or pypykatz can be used to crack out the passwords easily for us. This would let us gain the NT hash of the svc\_backup user.
6. From there, we can evil-winrm using PTH and get the user shell.
7. This user would have the SeRestore and SeBackup privilege, which can be used for copying multiple sensitive files. We can grab the root flag this way for CTF sake.
8. For Pentesting sake, we would have to use the Disk Shadow method, which would backup the entire disk to another drive. From there we can crack the NTDS.dit file and get out the NT hash for the Administrator. We can also grab the system registry to extract the hash using the secretsdump.py method.
9. PTH and authenticate as him, pwned.
