---
layout: post
title:  "Ansible Basic Series: Mengenal Variabel dan Facts"
comments: true
date:   2019-12-09
desc: "Ansible Basic Series"
keywords: "linux,basic,automation,ansible"
categories: [Linux]
tags: [ansible basic,linux,automation]
icon: icon-shell
---
Variabel dalam ansible terdiri dari 3 *scope level* yaitu:
* Global scope	: Terdapat di ansible.cfg
* Play scope	: Terdapat dalam playbook
* Host scope	: Terdapat dalam inventory, fact gathering atau task.

Jika terdapat variabel yang sama diantara ketiganya, maka alur overriding nya adalah host variabel>play variabel>global variabel. Misal terdapat variabel yang sama pada global dan host maka yang akan digunakan adalah variabel pada host.

### Pendefinisian Variabel pada Playbook

Pendefinisian variabel dapat dilakukan dengan mendefinisikannya langsung pada playbook atau melalui sebuah file variabel. Berikut perbedaannya:

#### Secara langsung

```
...

- hosts: all
    vars:
      user: riupie
      home: /home/riupie
  tasks:
    - name: Membuat user {{ user }}
      user:
        name: "{{ user }}"		# Jika variabel digunakan untuk memulai sebuah value maka harus diberi double quote
...

```
#### Secara tidak langsung

```
...

- hosts: all
    vars_files:
      - vars/users.yml
...

```
Lalu pada direktori vars dibuat file users.yml.
```
user: riupie
home: /home/riupie
```
### Pendefinisian variabel pada host
Pendefinisian variabel pada host dapat dilakukan dengan beberapa cara, misal dengan mendefinisikan langsung pada file inventory.

#### Untuk satu host

```
[servers]
demo.example.com  	ansible_user=riupie
```
#### Untuk group

```
[servers]
servera.rahmatawe.com
serverb.rahmatawe.com

[servers:vars]
ansible_user=riupie
```
#### Host Varibel dengan direktori group_vars dan host_vars

Untuk pendefinisian host variabel yang lebih rapi bisa dengan membuat direktori 
group_vars untuk menyimpan variabel sebuah grup dan direktori host_vars untuk menyimpan variabel spesifik suatu host. Lalu di dalam masing-masing direktori tersebut dibuat file-file dengan nama sesuai dengan nama grup atau host yang dimaksud. Selanjutnya didalam file tersebut barulah didefinisikan variabel-variabel nya. Berikut ini struktur dari file-file tersebut.

```
project
├── ansible.cfg
├── group_vars
│   │
│   ├── dbservers
│   │   
│   └── datacenters2
├── host_vars
│   │
│   ├── servera.rahmatawe.com
│   │
│   ├── serverb.rahmatawe.com
│   │
│   └── serverc.rahmatawe.com
├── inventory
└── playbook.yml

```

### Penulisan Variabel dengan Arrays
```
users:
  rahmat:
    first_name: Rahmat 
    last_name: Agung
    home_dir: /users/rahmat
  ari:
    first_name: Ari
    last_name: Wibowo
    home_dir: /users/ari
```
Untuk pemanggilan variabel diatas didalam playbook dapat dilakukan dengan cara sebagai berikut.

```
#Cara 1. Return Rahmat
users['rahmat']['first_name']

#Cara 2. Return Ari
users.ari.first_name
```

### Menyimpan Output Task

Output dari sebuah task dapat disimpan ke dalam sebuah variabel dengan menggunakan parameter *register*. Tujuan penggunaan biasanya untuk *debugging*.
```
---
- name: Installs webserver
  hosts: all
  tasks:
    - name: Install httpd
      yum:
        name: httpd
        state: installed
      register: install_output
    
    - debug: var=install_output
```
---
