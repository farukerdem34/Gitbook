# Resourced (WIP)

## Gaining Access

Nmap scan:

```
$ nmap -p- --min-rate 4000 -Pn 192.168.233.175
Starting Nmap 7.93 ( https://nmap.org ) at 2023-06-30 21:34 +08
Nmap scan report for 192.168.233.175
Host is up (0.17s latency).
Not shown: 65515 filtered tcp ports (no-response)
PORT      STATE SERVICE
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
3389/tcp  open  ms-wbt-server
5985/tcp  open  wsman
9389/tcp  open  adws
49666/tcp open  unknown
49667/tcp open  unknown
49670/tcp open  unknown
49671/tcp open  unknown
49703/tcp open  unknown
49776/tcp open  unknown
```

This is an AD machine.&#x20;
