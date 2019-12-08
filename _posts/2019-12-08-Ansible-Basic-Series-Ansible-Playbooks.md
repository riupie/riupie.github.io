---
layout: post
title:  "Ansible Basic Series: Menggunakan Ansible Playbooks"
comments: true
date:   2019-12-08
desc: "Ansible Basic Series"
keywords: "linux,basic,automation,ansible"
categories: [Linux]
tags: [ansible basic,linux,automation]
icon: icon-shell
---

### Basic Playbook

*Play* merupakan sebuah task yang berjalan pada sekumpulan host yang ada di inventory. Sedangkan playbook merupakan sebuah teks file yang berisi satu atau lebih *play* yang dijalankan dengan urutan yang spesifik. Berikut merupakan perbandingan antara Ansible Ad-hoc command dan hasil konversinya saat dijadikan Playbook.

#### Simple Ad-hoc Command

```
ansible -m user -a "name=riupie uid=1002 state=present" servera.rahmatawe.com
```

#### Simple Playbook sample.yml

```
---
- name: Penambahan user baru 				# Diskripsi mengenai Playbook
  hosts: servera.rahmatawe.com				# target host
  tasks:						# List ordered task yang akan dijalankan
    - name: Menambahkan user riupie dengan uid 1002	# keterangan mengenai task (optional)
      user:						# module yang akan dijalan oleh task
        name: riupie        				#atribut-atribut dari module
        uid: 1002
        state: present
```

Sebuah file playbook akan disimpan format YAML. Dalam file YAML, dalam setiap pembuatan file biasanya akan diawali *document marker* berupa (---). Selanjutnya, untuk mengecek file playbook yang dibuat sudah benar (tidak terdapat kesalahan indentasi, dll) dapat menggunakan perintah berikut:

```
ansible-playbook --syntax-check sample.yml
```

Selain itu, kita juga bisa melakukan *dry run* untuk melihat perubahan apa saja yang akan dilakukan saat suatu playbook dijalankan.

```
ansible-playbook -C sample.yml
```
### Multiple Plays Playbook

Berikut ini merupakan contoh Playbook yang terdiri dari 2 play.

```
---
- name: Enable internet services
  hosts: serverb.lab.example.com
  become: yes
  tasks:
  - name: Install package
    yum:
      name:
        - firewalld
        - httpd
        - mariadb-server
        - php
        - php-mysqlnd
      state: latest
  - name: Firewalld is running
    service:
      name: firewalld
      enabled: yes
      state: started
  - name: Allow httpd on firewalld
    firewalld:
      service: http
      state: enabled
      permanent: yes
      immediate: yes
  - name: Ensure http running
    service:
      name: httpd
      enabled: true
      state: started
  - name: Ensure mariadb running
    service:
      name: mariadb
      enabled: true
      state: started
  - name: Download source code
    get_url:
      url: https://github.com/google/web-starter-kit/blob/master/app/index.html
      dest: /var/www/html
      mode: 0644

- name: Test internet
  hosts: localhost
  become: no
  tasks:
    - name: Check web serverb
      uri:
        url: http://serverb.riupie.com
        status_code: 200
```

---
