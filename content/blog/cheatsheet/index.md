---
draft: false
resources: []
title: cheatsheet
date: '2021-03-01'
categories:
  - Cheatsheet
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
---
