---
draft: true
resources: []
title: Ceph Data Distribution using Crush Map Rule
date: now
categories:
  - cloud
description: ''
series: []
tags:
  - ceph
  - linux
  - storage

---

## Ceph verify
```bash
root@ag-ceph1:~/ceph-manual# ceph osd tree
ID   CLASS  WEIGHT   TYPE NAME          STATUS  REWEIGHT  PRI-AFF
 -1         0.31177  root default                                
 -3         0.03897      host ag-ceph1                           
  0    hdd  0.01949          osd.0          up   1.00000  1.00000
  1    hdd  0.01949          osd.1          up   1.00000  1.00000
 -5         0.03897      host ag-ceph2                           
  2    hdd  0.01949          osd.2          up   1.00000  1.00000
  3    hdd  0.01949          osd.3          up   1.00000  1.00000
 -7         0.03897      host ag-ceph3                           
  4    hdd  0.01949          osd.4          up   1.00000  1.00000
  5    hdd  0.01949          osd.5          up   1.00000  1.00000
 -9         0.03897      host ag-ceph4                           
  6    hdd  0.01949          osd.6          up   1.00000  1.00000
  7    hdd  0.01949          osd.7          up   1.00000  1.00000
-11         0.03897      host ag-ceph5                           
  8    hdd  0.01949          osd.8          up   1.00000  1.00000
  9    hdd  0.01949          osd.9          up   1.00000  1.00000
-13         0.03897      host ag-ceph6                           
 10    hdd  0.01949          osd.10         up   1.00000  1.00000
 11    hdd  0.01949          osd.11         up   1.00000  1.00000
-15         0.03897      host ag-ceph7                           
 12    hdd  0.01949          osd.12         up   1.00000  1.00000
 13    hdd  0.01949          osd.13         up   1.00000  1.00000
-17         0.03897      host ag-ceph8                           
 14    hdd  0.01949          osd.14         up   1.00000  1.00000
 15    hdd  0.01949          osd.15         up   1.00000  1.00000
```


## Create Bucket
```bash
ceph osd crush add-bucket rack01 rack
ceph osd crush add-bucket rack02 rack
ceph osd crush add-bucket rack03 rack
ceph osd crush add-bucket rack04 rack


#Move rack

ceph osd crush move rack01 root=default
ceph osd crush move rack02 root=default
ceph osd crush move rack03 root=default
ceph osd crush move rack04 root=default

```

## Ceph verify
```bash

root@ag-ceph1:~# ceph osd tree
ID   CLASS  WEIGHT   TYPE NAME          STATUS  REWEIGHT  PRI-AFF
 -1         0.31177  root default                                
 -3         0.03897      host ag-ceph1                           
  0    hdd  0.01949          osd.0          up   1.00000  1.00000
  1    hdd  0.01949          osd.1          up   1.00000  1.00000
 -5         0.03897      host ag-ceph2                           
  2    hdd  0.01949          osd.2          up   1.00000  1.00000
  3    hdd  0.01949          osd.3          up   1.00000  1.00000
 -7         0.03897      host ag-ceph3                           
  4    hdd  0.01949          osd.4          up   1.00000  1.00000
  5    hdd  0.01949          osd.5          up   1.00000  1.00000
 -9         0.03897      host ag-ceph4                           
  6    hdd  0.01949          osd.6          up   1.00000  1.00000
  7    hdd  0.01949          osd.7          up   1.00000  1.00000
-11         0.03897      host ag-ceph5                           
  8    hdd  0.01949          osd.8          up   1.00000  1.00000
  9    hdd  0.01949          osd.9          up   1.00000  1.00000
-13         0.03897      host ag-ceph6                           
 10    hdd  0.01949          osd.10         up   1.00000  1.00000
 11    hdd  0.01949          osd.11         up   1.00000  1.00000
-15         0.03897      host ag-ceph7                           
 12    hdd  0.01949          osd.12         up   1.00000  1.00000
 13    hdd  0.01949          osd.13         up   1.00000  1.00000
-17         0.03897      host ag-ceph8                           
 14    hdd  0.01949          osd.14         up   1.00000  1.00000
 15    hdd  0.01949          osd.15         up   1.00000  1.00000
-19               0      rack rack01                             
-20               0      rack rack02                             
-21               0      rack rack03                             
-22               0      rack rack04 
```

## Ceph crush
```bash
ceph osd crush move ag-ceph1  rack=rack01
ceph osd crush move ag-ceph2  rack=rack01
ceph osd crush move ag-ceph3  rack=rack02
ceph osd crush move ag-ceph4  rack=rack02
ceph osd crush move ag-ceph5  rack=rack03
ceph osd crush move ag-ceph6  rack=rack03
ceph osd crush move ag-ceph7  rack=rack04
ceph osd crush move ag-ceph8  rack=rack04
```

```bash
root@ag-ceph1:~# ceph osd tree
ID   CLASS  WEIGHT   TYPE NAME              STATUS  REWEIGHT  PRI-AFF
 -1         0.31177  root default                                    
-19         0.07794      rack rack01                                 
 -3         0.03897          host ag-ceph1                           
  0    hdd  0.01949              osd.0          up   1.00000  1.00000
  1    hdd  0.01949              osd.1          up   1.00000  1.00000
 -5         0.03897          host ag-ceph2                           
  2    hdd  0.01949              osd.2          up   1.00000  1.00000
  3    hdd  0.01949              osd.3          up   1.00000  1.00000
-20         0.07794      rack rack02                                 
 -7         0.03897          host ag-ceph3                           
  4    hdd  0.01949              osd.4          up   1.00000  1.00000
  5    hdd  0.01949              osd.5          up   1.00000  1.00000
 -9         0.03897          host ag-ceph4                           
  6    hdd  0.01949              osd.6          up   1.00000  1.00000
  7    hdd  0.01949              osd.7          up   1.00000  1.00000
-21         0.07794      rack rack03                                 
-11         0.03897          host ag-ceph5                           
  8    hdd  0.01949              osd.8          up   1.00000  1.00000
  9    hdd  0.01949              osd.9          up   1.00000  1.00000
-13         0.03897          host ag-ceph6                           
 10    hdd  0.01949              osd.10         up   1.00000  1.00000
 11    hdd  0.01949              osd.11         up   1.00000  1.00000
-22         0.07794      rack rack04                                 
-15         0.03897          host ag-ceph7                           
 12    hdd  0.01949              osd.12         up   1.00000  1.00000
 13    hdd  0.01949              osd.13         up   1.00000  1.00000
-17         0.03897          host ag-ceph8                           
 14    hdd  0.01949              osd.14         up   1.00000  1.00000
 15    hdd  0.01949              osd.15         up   1.00000  1.00000
```