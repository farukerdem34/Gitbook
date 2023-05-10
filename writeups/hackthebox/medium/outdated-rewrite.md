# Outdated (WIP)

## Gaining Access

Nmap scan:

```
$ nmap -p- --min-rate 5000 -Pn 10.129.193.194
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-10 13:54 EDT
Nmap scan report for 10.129.193.194
Host is up (0.012s latency).
Not shown: 65512 filtered tcp ports (no-response)
PORT      STATE SERVICE
25/tcp    open  smtp
53/tcp    open  domain
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
5985/tcp  open  wsman
8530/tcp  open  unknown
8531/tcp  open  unknown
9389/tcp  open  adws
49667/tcp open  unknown
49689/tcp open  unknown
49691/tcp open  unknown
49693/tcp open  unknown
49950/tcp open  unknown
49961/tcp open  unknown
59064/tcp open  unknown
```

Lots of AD ports. WIP.
