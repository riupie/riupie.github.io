---
draft: false
resources: []
title: cheatsheet
date: '2021-03-01'
categories:
  - cheatsheet
description: Command Cheatsheet Collection
series: []
tags:
  - linux

---

##### Running short-lived container to compile source code
```bash
$ sudo docker run --rm -it -v ${PWD}:/app -w /app openjdk:11-jdk-slim /bin/sh -c ./gradlew buildRun
```
   
##### Rsync over SSH on Spesific Port

```bash
$ rsync -avzhe 'ssh -p [port_number]' -P [user@remote_ip]:/path/to/file/source /path/to/file/destination
```
   
##### Scan IP on Network

```bash
$ nmap -sP [ip_address/prefix]
```
   
##### Search Available Version of Spesific Package on Debian Based OS

```bash
$ apt list -a <package_name>
```
   
##### Install Spesific Kubernetes Version

```bash
#example version 1.15.x#
$ sudo apt update;sudo apt install -qy kubelet=1.15.5-00 kubectl=1.15.5-00 kubeadm=1.15.5-00
```
   
##### Open Spesific Port on Centos 7

```bash
iptables -I INPUT 5 -p tcp -m state --state NEW -m tcp --dport 9100 -j ACCEPT
```
   
##### Replace String Using Sed

```bash
sed -i 's/old-string/new-string/g'
```

##### Find and copy files
```bash
find /var/cache/dnf -iname "*.rpm" -exec cp {} packages/ \;
```

##### Encrypt secret envar value using Google KMS
```bash
echo -n "This is my secret" | gcloud kms encrypt --plaintext-file=- --ciphertext-file=- --location=global --keyring=mykeyring --key=myappkey | base64 -w 0
```

##### Encrypt file using Google KMS
```bash
gcloud kms encrypt \
    --key myappkey \
    --keyring mykeyring \
    --location global  \
    --plaintext-file application.properties \
    --ciphertext-file application.properties.enc
```
##### Check DNS record
```bash
$ dig +noall +answer google.com

google.com.		204	IN	A	74.125.24.102
google.com.		204	IN	A	74.125.24.138
google.com.		204	IN	A	74.125.24.139
google.com.		204	IN	A	74.125.24.101
google.com.		204	IN	A	74.125.24.100
google.com.		204	IN	A	74.125.24.113
```

##### Check service port
```bash
$ getent services 53

domain                53/tcp
```
---
