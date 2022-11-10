# Compromised

Compromised

Friday, 25 March 2022

2:20 pm

A windows machine that requires some investigation of the Documents folder using SMB to get the profile.ps1 file, which contains the password for user 'scripting' in base64.

&#x20;

!\[\[102\_Compromised\_image001.png]]

&#x20;

From there, we discover we are part of the windows log event viewer. From there, we are able to execute a ps1 code within C:\Troubleshoot and get a .csv file. Within that file lies a huge chunk of obfuscated code.

&#x20;

&#x20;

&#x20;

&#x20;

!\[\[102\_Compromised\_image002.png]]

&#x20;

This is a mix of base64 and ASCII encoding, and when decrypted the script looks like this:

{width="12.177083333333334in" height="3.625in"}

&#x20;

This would indicate that there is some kind of password going around. The first base64 string does not seem to be decodable with just base64 and is actually the hidden password. From there, we take parts of the string, particularly the $ms variable. Look at the important parts, then extract them to run with Kali powershell.

&#x20;

!\[\[102\_Compromised\_image004.png]]

&#x20;

From there we can evil-winrm in as the administrator with this password.

&#x20;

!\[\[102\_Compromised\_image005.png]]

&#x20;

Not exactly very OSCP like, more of a challenge of finding stuff.

&#x20;

&#x20;
