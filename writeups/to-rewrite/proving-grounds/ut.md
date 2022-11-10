# UT

UT99

Thursday, 17 March 2022

4:26 pm

From the name, there seems to be an exploit regarding it already with Unreal Tournament.

&#x20;

!\[\[91\_UT99\_image001.png]]

Run that exploit, as well as generate our own form of shell code and then run it to get a reverse shell. No need to bring out the debugger for this one yet.

&#x20;

Within the ftp folder, there's an application that suffers from an unquoted service path, meaning we can replace the exe with our own to execute code as the administrator.

&#x20;

After replacing it with Foxit.exe, reboot it and wait. The shell will come.

!\[\[91\_UT99\_image002.png]]

&#x20;

&#x20;

&#x20;

&#x20;
