---
layout: post
title:  "Ansible Basic Series: Menggunakn Ad Hoc Command"
comments: true
date:   2019-12-08
desc: "Ansible Basic Series"
keywords: "linux,basic,automation,ansible"
categories: [Linux]
tags: [ansible basic,linux,automation]
icon: icon-shell
---
Ad-hoc command merupakan perintah otomasi pada Ansible yang hanya menggunakan satu baris perintah, misal :

```
ansible all -m ping
```
Perintah diatas akan menjalankan perintah ping ke semua node dalam inventory ansible. Adapun komponen paling dasar untuk menjalankan dan menggunakan Ad Hoc command pada Ansible yaitu:
* ansible.cfg
* inventory file
* Ansible itu sendiri

#### ansible.cfg

File ini berisi konfigurasi ansible seperti user apa yang akan digunakan untuk eksekusi di remote host, bagaimana metode eksekusi perintahnya dan inventory mana yang akan digunakan. Adapun format dari ansible.cfg adalah sebagai berikut:

```
[defaults]
inventory=inventory/inventory
remote_user=riupie
[privilege_escalation]
become=false
become_user=root
become_method=sudo
become_ask_pass=false
```
By default, saat perintah ansible dijalankan, ansible akan mengecek pada working directory apakah ada file ansible.cfg. Jika tidak ada, maka pengecekan akan dilakukan di ~/.ansible.cfg dan terakhir akan mengecek default konfigurasi pada /etc/ansible/ansible.cfg.

#### Inventory
Inventory merupakan file yang berisi kumpulan host-host yang di-manage oleh Ansible. Adapun format nya sebagai berikut:

```
[datacenter:children]
webserver
dbserver
[webserver]
web[1:3].myserver.com
[dbserver]
192.168.1.[10:200]
[dns]
[a:d].mydns.com
```
#### Contoh penggunaan
Menambahkan MOTD ke semua webserver yang kita manage.

```
ansible webserver -m copy -a 'content="Warning! Authorized Access Only!" dest=/etc/motd' --become
```
Option -a digunakan untuk menambahkan parameter-parameter tertentu sesuai dengan module yang sedang digunakan. Sedangkan option --become digunakan agar eksekusi perintah dijalankan sebagai root. Lalu untuk mengecek apakah teks yang sudah kita eksekusi sudah masuk ke file /etc/motd dapat menggunakan command ad-hoc berikut:

```
ansible webserver -m command -a 'cat /etc/motd'
```
Lalu, bagaiamana caranya untuk mengetahui parameter-parameter yang tersedia untuk suatu modul? Gunakan perintah berikut:

```
ansible-doc -s [modul_name]
```

---
