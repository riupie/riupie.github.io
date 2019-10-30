---
layout: page
title:  "Cheatsheet"
desc: "All cheatsheet of my working use case"
keywords: "linux, storage, networking, security"
comments:false
permalink: /cheatsheet
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
