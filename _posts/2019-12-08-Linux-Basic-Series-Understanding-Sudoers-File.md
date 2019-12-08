---
layout: post
title:  "Linux Basic Series: Memahami Sudoers File"
comments: true
date:   2019-12-08
desc: "Linux Basic Series"
keywords: "linux,basic"
categories: [Linux]
tags: [linux basic]
icon: icon-shell
---
Dalam sistem operasi berbasis Linux agar seorang user dapat mengekseskusi perintah-perintah level admin/root, user tersebut perlu diberi akses `sudo` terhadap root. Perintah sudo (superuser do) berfungsi untuk mengijinkan seorang user untuk menjalankan sebuah program sebagai user lain (yang dalam kebanyakan kasus sebagai user root). Ada beberapa cara untuk memberi akses sudo kepada user diantaranya yaitu memasukkan user tersebut ke group sudo (Debian based) atau wheel (Centos & RHEL) dan menambahkan ke sudoer file.

Untuk cara yang kedua bisa dilakukan dengan menambahkan beberapa syntax ke direktori /etc/sudoers.d/. Adapun format syntax nya sendiri seperti berikut:

```
USER/GROUP HOST_COMPUTER= (USER:GROUP) COMMAND   
```

Jadi, semisal dalam sudoer file terdapat syntax berikut:

```
riupie ALL=(ALL:ALL) ALL
%wheel ALL=(ALL:ALL) ALL
```

Syntax pertama dapat dibaca: User riupie dapat menjalankan perintah sudo di komputer manapun sebagai user apapun dan group apapun dan perintah apapaun. Syntax kedua hampir sama, hanya saja tanda (%) mengindikasikan obyeknya adalah group. Lalu bagaimana penggunaannya? Pemberian akses pada user riupie diatas membuat user riupie dapat menjalankan perintah berikut:

```
sudo -u andy -g admin id
```
User riupie dapat menjalankan perintah `id` sebagai user andy pada grup admin. Seringkali, kita juga akan menemui format syntax berikut:

```
riupie ALL=(ALL:ALL) NOPASSWD: ALL
```
Hal ini berarti bahwa user riupie tidak perlu memasukkan password ketika menjalankan perintah sudo.
---

