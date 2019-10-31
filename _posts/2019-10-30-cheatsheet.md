---
layout: post
title:  "Cheatsheet"
comments: false
date:   2019-10-30
desc: "All cheatsheet of my working use case"
keywords: "linux, storage, networking, security"
permalink: /cheatsheet
icon: icon-shell
---


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
---
