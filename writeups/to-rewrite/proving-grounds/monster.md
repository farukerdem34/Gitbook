# Monster

Monster

Friday, 18 March 2022

4:54 pm

Windows machine with an easy web vuln of monstra 3.0.4 found after a quick gobuster revealing a hidden directory of a blog.

!\[\[95\_Monster\_image001.png]]

&#x20;

Crack the hash and determine that this is using a Mike14 password.

&#x20;

For privilege escalation, determine that xampp is running an older version that can be used for it.

&#x20;

Hence, we are able to include malicious conn.bat files which would add us into the administrator group as admin.

&#x20;

This rises because Xampp-service can be run as the low privileged user and can perform specific administrator actions.

&#x20;

[https://github.com/z3n70/CVE-2021-41277](https://github.com/z3n70/CVE-2021-41277)

Read full POC here.

!\[\[95\_Monster\_image002.png]]

Now that we are an administrator, we can restart the device allow all users to view the administrator account.and get the proof.txt.

&#x20;

!\[\[95\_Monster\_image003.png]]

&#x20;
