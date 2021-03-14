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

##### Running short-lived container to compile source code
```bash
sudo docker run --rm -it -v ${PWD}:/app -w /app openjdk:11-jdk-slim /bin/sh -c ./gradlew buildRun
```
