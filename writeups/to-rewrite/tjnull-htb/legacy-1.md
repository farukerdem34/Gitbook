# Legacy

Legacy

Tuesday, 1 February 2022

1:18 am

**\[Legacy]{.underline}**

Nmap scan report for 10.10.10.4

Host is up (0.010s latency).

Not shown: 997 filtered ports

PORT     STATE  SERVICE

139/tcp  open   netbios-ssn

445/tcp  open   microsoft-ds

3389/tcp closed ms-wbt-server

Nmap scan report for 10.10.10.4

Host is up (0.012s latency).

Not shown: 65532 filtered ports

PORT     STATE  SERVICE       VERSION

139/tcp  open   netbios-ssn   Microsoft Windows netbios-ssn

445/tcp  open   microsoft-ds  Microsoft Windows XP microsoft-ds

3389/tcp closed ms-wbt-server

No SMB shares revealed.&#x20;

Take note we can use --script vuln for nmap in order to reveal what are the vulnerable things within a host.

We can do this two ways, we can do either the Metasploit way or the manual way. I did it the MSF way.
