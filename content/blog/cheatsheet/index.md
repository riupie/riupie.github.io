---
draft: false
resources: []
title: Cheatsheet
date: '2021-03-01'
categories:
  - Cheatsheet
description: Command Cheatsheet Collection
series: []
tags:
  - linux

---

#### Running short-lived container to compile source code
```bash
sudo docker run --rm -it -v ${PWD}:/app -w /app openjdk:11-jdk-slim /bin/sh -c ./gradlew buildRun
```

#### Rsync over SSH on Spesific Port

```
# rsync -avzhe 'ssh -p [port_number]' -P [source]:/path/to/file [destination]:/path/to/file
```


#### Scan IP on Network

```
$ nmap -sP [ip_address/prefix]
```
#### Copy Public Key to Other Server

```
$ ssh-copy-id [usernmae]@[remote_server]
```

#### Search Available Version of Spesific Package on Debian Based OS

```
$ apt list -a <package_name>
```

#### Install Spesific Kubernetes Version

```
#example version 1.15.x#
$ sudo apt update;sudo apt install -qy kubelet=1.15.5-00 kubectl=1.15.5-00 kubeadm=1.15.5-00
```
### Open Spesific Port on Centos 7

```
iptables -I INPUT 5 -p tcp -m state --state NEW -m tcp --dport 9100 -j ACCEPT
```
### Replace String Using Sed

```
sed -i 's/old-string/new-string/g'
```
---
