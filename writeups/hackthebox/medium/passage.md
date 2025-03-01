# Passage

## Gaining Access

Nmap scan:

<figure><img src="../../../.gitbook/assets/image (119) (3).png" alt=""><figcaption></figcaption></figure>

### Passage News

Port 80 reveals some kind of website archive thing:

<figure><img src="../../../.gitbook/assets/image (109) (4).png" alt=""><figcaption></figcaption></figure>

Checking the page source, we find that this is running CuteNews, which had a few RCE exploits available:

{% embed url="https://www.exploit-db.com/exploits/48800" %}

<figure><img src="../../../.gitbook/assets/image (104) (1).png" alt=""><figcaption></figcaption></figure>

With this, we can easily gain a reverse shell:

<figure><img src="../../../.gitbook/assets/image (135) (2).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

### Paul Credentials

Within the `/var/www/html/CuteNews/cdata/users` directory, we can find some base64 encoded lines:

<figure><img src="../../../.gitbook/assets/image (80) (4).png" alt=""><figcaption></figcaption></figure>

When one of them was decoded, we find a token of some sorts:

<figure><img src="../../../.gitbook/assets/image (121) (1).png" alt=""><figcaption></figcaption></figure>

We can crack this hash on crackstation:

<figure><img src="../../../.gitbook/assets/image (123) (2) (1).png" alt=""><figcaption></figcaption></figure>

Then we can `su` to `paul`:

<figure><img src="../../../.gitbook/assets/image (89) (4).png" alt=""><figcaption></figcaption></figure>

Cool

### SSH to Nadav

When I ran LinPEAS on the machine, I found that the public key of `nadav` was the public key of `paul`...?

<figure><img src="../../../.gitbook/assets/image (83) (1) (4).png" alt=""><figcaption></figcaption></figure>

I tried to `ssh` in as `nadav` from `paul`, and it worked!

<figure><img src="../../../.gitbook/assets/image (116) (3).png" alt=""><figcaption></figcaption></figure>

### USBCreator

When running another LinPEAS, we find this part here:

<figure><img src="../../../.gitbook/assets/image (131) (1).png" alt=""><figcaption></figcaption></figure>

{% embed url="https://rioasmara.com/2021/07/16/usbcreator-d-bus-privilege-escalation/" %}

{% code overflow="wrap" %}
```bash
gdbus call --system --dest com.ubuntu.USBCreator --object-path /com/ubuntu/USBCreator --method com.ubuntu.USBCreator.Image /root/.ssh/id_rsa /tmp/id_rsa true
```
{% endcode %}

Following this PoC would extract the private SSH key of `root` and allow me to SSH in as `root`:

<figure><img src="../../../.gitbook/assets/image (115) (3) (1).png" alt=""><figcaption></figcaption></figure>
