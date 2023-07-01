# Jacko (WIP)

## Gaining Access

Nmap scan:

```
$ nmap -p- --min-rate 4000 192.168.175.66
Starting Nmap 7.93 ( https://nmap.org ) at 2023-07-01 21:07 +08
Nmap scan report for 192.168.175.66
Host is up (0.17s latency).
Not shown: 65520 closed tcp ports (conn-refused)
PORT      STATE    SERVICE
80/tcp    open     http
135/tcp   open     msrpc
139/tcp   open     netbios-ssn
445/tcp   open     microsoft-ds
5040/tcp  open     unknown
8082/tcp  open     blackice-alerts
9092/tcp  open     XmlIpcRegSvc
41213/tcp filtered unknown
49664/tcp open     unknown
49665/tcp open     unknown
49666/tcp open     unknown
49667/tcp open     unknown
49668/tcp open     unknown
49669/tcp open     unknown
57199/tcp filtered unknown
```

WIP!
