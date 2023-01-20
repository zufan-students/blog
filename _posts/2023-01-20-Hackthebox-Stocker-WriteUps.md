---
title: "HackTheBox - Stocker WriteUps"
date: 2023-01-20 13:00:00 +07:00
categories: [WriteUps]
tags: [ctf, hackthebox, pentest, writeups]
image: 
---

# HackTheBox Stocker Writeups

Scanning dengan Nmap, terdapat port terbuka yaitu 22 SSH dan 80 HTTP: 
![image](https://media.discordapp.net/attachments/740245586095112242/1065882948445929482/image.png)

Tampilan port 80 menampilkan halaman page:
![image](https://media.discordapp.net/attachments/740245586095112242/1065883097834475571/image.png)

Kita disini fuzzing subdomain dengan wfuzz dan mendapatkan subdomain:
![image](https://media.discordapp.net/attachments/740245586095112242/1065883346493767740/image.png)

Berikut adalah tampilan subdomain yang didapat terdapat login panel:
![image](https://media.discordapp.net/attachments/740245586095112242/1065883481642639430/image.png)

Setelah mencoba beberapa hal kita menemukan bahwa login panel ini bisa kita bypass dengan NoSQL yang didapat dari reference [hacktricks](https://book.hacktricks.xyz/pentesting-web/nosql-injection#basic-authentication-bypass) dengan payload:
```
{"username": {"$ne": null}, "password": {"$ne": null} }
```
Dan mengubah
```
Content-Type: application/json
```

Setelah mengubahnya dengan payload kita akan mendapatkan response error syntax error, dan disini juga menampilkan ```Path Disclosure``` letak dimana website ini berjalan.
![image](https://media.discordapp.net/attachments/740245586095112242/1065884013463613490/image.png)

Disini bila kita merefresh website kita akan memasukin pada tampilan seperti Digambar. Bahwasanya website ini rentan terhadap Bypass NoSQL Injection
![image](https://media.discordapp.net/attachments/740245586095112242/1065884323686912090/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1065884344599728198/image.png)

Disini kita mencoba menambah quatinty pada stok
![image](https://media.discordapp.net/attachments/740245586095112242/1065884465299193876/image.png)

Mendapatkan response berhasil  setelah kita purchase. Disini menadapatkan Order ID dan Informasi struk yang telah dibeli
![image](https://media.discordapp.net/attachments/740245586095112242/1065884605628031036/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1065884802881961994/image.png)

Kemudian kita mencoba menyambungkan pada Intercept Burp Suite yang sebelumnya kita purchase mengarah pada API
![image](https://media.discordapp.net/attachments/740245586095112242/1065884973078421545/image.png)

Setelah itu kita mencoba menyisipkan payload LFI yang disambungkan dengan Cross Site Scripting
![image](https://media.discordapp.net/attachments/740245586095112242/1065885136509489202/image.png)

Dengan kerentanan LFI ini kita mendapatkan informasi yang terdapat pada server dengan mengganti Order ID pada payload API
![image](https://media.discordapp.net/attachments/740245586095112242/1065885312385024020/image.png)

Setelah mencari cari dengan kerentanan LFI, Kita disini mencoba mengubah payload lagi dengan membaca file ```index.js``` dan disini mendapatkan informasi sensitive pada ```mongodb```
![image](https://media.discordapp.net/attachments/740245586095112242/1065885578199060561/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1065885592698753095/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1065885623539470337/image.png)

Ternyata informasi password pada ```mongodb``` ini bisa kita login dengan password yang didapat tadi.
![image](https://media.discordapp.net/attachments/740245586095112242/1065886055250804787/image.png)

Setelah itu membaca sudoers dengan ```sudo -l``` dan mendapatkan bila kita menjalankan file ```.js``` kita dapat langsung memiliki akses privileges dan kita bisa langsung mendapatkan root access
![image](https://media.discordapp.net/attachments/740245586095112242/1065886254568316978/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1065886419593207838/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1065886443127447652/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1065886460428943510/image.png?width=668&height=607)
