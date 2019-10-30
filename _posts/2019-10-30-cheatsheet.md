---
layout: post
title:  "Cheatsheet"
comments:false
date:   2017-12-21
desc: "All cheatsheet of my working use case"
keywords: "linux, storage, networking, security"
categories: [page]
tags: [linux]
icon: icon-html
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
ssh-copy-id [usernmae]@[remote_server]
```
---

