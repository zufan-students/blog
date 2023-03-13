---
title: "Bypass SSL Pinning with Burp Suite"
date: 2023-03-13 02:00:00 +07:00
categories: [WriteUps]
tags: [tricks, bypass, tools, writeups]
image: 
---

# Bypass SSL Pinning with Burp Suite

Haii!! How are you?!!

Kali ini saya akan membagikan artikel tentang bagaimana cara Bypass SSL Pinning di Android Emulator Genymotion dengan Burp Suite. oke langsung saja. 

## Frida
Frida adalah sebuah tool dynamic instrumentation yang sangat berguna untuk melakukan debugging, tracing, dan memodifikasi aplikasi pada platform mobile dan desktop. Frida bekerja pada berbagai platform, termasuk Android, iOS, Windows, macOS, dan Linux. Pada artikel ini, saya akan memberikan panduan cara install Frida pada sistem operasi Kali Linux.

Sebelum kita memulai, pastikan sistem operasi Kali Linux yang kamu gunakan sudah di-update terlebih dahulu. Kita juga perlu menginstal Python dan pip. Untuk meng-update sistem operasi Kali Linux, jalankan perintah berikut di terminal:

```sql
sudo apt-get update
sudo apt-get upgrade
```

Untuk menginstal Python dan pip, jalankan perintah berikut:

```sql
sudo apt-get install python3
sudo apt-get install python3-pip
```

Setelah itu, kita bisa mulai menginstal Frida pada Kali Linux dengan mengikuti langkah-langkah di bawah ini:

1.	Install dependensi yang dibutuhkan

Pertama-tama, kita perlu menginstal beberapa dependensi yang dibutuhkan oleh Frida. Jalankan perintah berikut di terminal untuk menginstal dependensi tersebut:

```sql
sudo apt-get install build-essential git python3-dev libffi-dev libssl-dev
```

2.	Instal Frida

Setelah dependensi terinstal, kita bisa menginstal Frida. Terdapat dua cara untuk menginstal Frida, yaitu menggunakan pip atau mengunduh Frida secara langsung dari situs web resminya.

2.1 Instal Frida menggunakan pip

Untuk menginstal Frida menggunakan pip, jalankan perintah berikut di terminal:

```sql
sudo pip3 install frida
```

2.2. Instal Frida dengan mengunduh dari situs web resmi

Jika kamu ingin mengunduh Frida secara langsung dari situs web resminya, kamu bisa melakukan langkah-langkah berikut:

1.	Kunjungi situs web https://frida.re/ dan unduh Frida sesuai dengan sistem operasi dan arsitektur yang kamu gunakan. Misalnya, jika kamu menggunakan Kali Linux 64-bit, unduh file frida-server-x.x.x-linux-x86_64.tar.xz.
2.	Ekstrak file yang telah diunduh ke direktori yang kamu inginkan.
3.	Jalankan Frida dengan menggunakan perintah berikut:

```bash
./frida-server
```

Jika kamu menginstal Frida menggunakan metode kedua, pastikan bahwa file frida-server yang telah diunduh berada di direktori yang sama dengan direktori saat kamu menjalankan perintah di atas.

Setelah selesai menginstal, kamu bisa memeriksa apakah Frida sudah terinstal dengan baik atau tidak dengan menjalankan perintah berikut:

```sql
frida --version
```
![image](https://media.discordapp.net/attachments/740245586095112242/1084756776131502131/image.png)

Jika Frida sudah terinstal dengan baik, maka versi Frida akan ditampilkan di terminal.

Selanjutnya, download script yang akan digunakan di https://codeshare.frida.re/@pcipolloni/universal-android-ssl-pinning-bypass-with-frida/.

```js
/* 
   Android SSL Re-pinning frida script v0.2 030417-pier 

   $ adb push burpca-cert-der.crt /data/local/tmp/cert-der.crt
   $ frida -U -f it.app.mobile -l frida-android-repinning.js --no-pause

   https://techblog.mediaservice.net/2017/07/universal-android-ssl-pinning-bypass-with-frida/
   
   UPDATE 20191605: Fixed undeclared var. Thanks to @oleavr and @ehsanpc9999 !
*/

setTimeout(function(){
    Java.perform(function (){
    	console.log("");
	    console.log("[.] Cert Pinning Bypass/Re-Pinning");

	    var CertificateFactory = Java.use("java.security.cert.CertificateFactory");
	    var FileInputStream = Java.use("java.io.FileInputStream");
	    var BufferedInputStream = Java.use("java.io.BufferedInputStream");
	    var X509Certificate = Java.use("java.security.cert.X509Certificate");
	    var KeyStore = Java.use("java.security.KeyStore");
	    var TrustManagerFactory = Java.use("javax.net.ssl.TrustManagerFactory");
	    var SSLContext = Java.use("javax.net.ssl.SSLContext");

	    // Load CAs from an InputStream
	    console.log("[+] Loading our CA...")
	    var cf = CertificateFactory.getInstance("X.509");
	    
	    try {
	    	var fileInputStream = FileInputStream.$new("/data/local/tmp/cert-der.crt");
	    }
	    catch(err) {
	    	console.log("[o] " + err);
	    }
	    
	    var bufferedInputStream = BufferedInputStream.$new(fileInputStream);
	  	var ca = cf.generateCertificate(bufferedInputStream);
	    bufferedInputStream.close();

		var certInfo = Java.cast(ca, X509Certificate);
	    console.log("[o] Our CA Info: " + certInfo.getSubjectDN());

	    // Create a KeyStore containing our trusted CAs
	    console.log("[+] Creating a KeyStore for our CA...");
	    var keyStoreType = KeyStore.getDefaultType();
	    var keyStore = KeyStore.getInstance(keyStoreType);
	    keyStore.load(null, null);
	    keyStore.setCertificateEntry("ca", ca);
	    
	    // Create a TrustManager that trusts the CAs in our KeyStore
	    console.log("[+] Creating a TrustManager that trusts the CA in our KeyStore...");
	    var tmfAlgorithm = TrustManagerFactory.getDefaultAlgorithm();
	    var tmf = TrustManagerFactory.getInstance(tmfAlgorithm);
	    tmf.init(keyStore);
	    console.log("[+] Our TrustManager is ready...");

	    console.log("[+] Hijacking SSLContext methods now...")
	    console.log("[-] Waiting for the app to invoke SSLContext.init()...")

	   	SSLContext.init.overload("[Ljavax.net.ssl.KeyManager;", "[Ljavax.net.ssl.TrustManager;", "java.security.SecureRandom").implementation = function(a,b,c) {
	   		console.log("[o] App invoked javax.net.ssl.SSLContext.init...");
	   		SSLContext.init.overload("[Ljavax.net.ssl.KeyManager;", "[Ljavax.net.ssl.TrustManager;", "java.security.SecureRandom").call(this, a, tmf.getTrustManagers(), c);
	   		console.log("[+] SSLContext initialized with our custom TrustManager!");
	   	}
    });
},0);
```

Setelah itu, salin script yang terdapat pada link tersebut ke dalam notepad atau text editor lainnya dan simpan dengan nama ```SSLPinningBypass.js```.
![image](https://media.discordapp.net/attachments/740245586095112242/1084757121754742784/image.png)

## Install ADB di Kali Linux

Pertama-tama, pastikan bahwa sistem Kali Linux Anda sudah diperbarui dengan repositori terbaru. Anda dapat melakukan ini dengan menjalankan perintah berikut di terminal:
```sql
sudo apt update
```

Setelah repositori diperbarui, Anda dapat menginstal ADB dengan menjalankan perintah berikut di terminal:
```sql
sudo apt install adb
```

Setelah instalasi selesai, Anda dapat memverifikasi apakah ADB telah berhasil diinstal dengan menjalankan perintah berikut di terminal:

```
adb version
```
![image](https://media.discordapp.net/attachments/740245586095112242/1084757677739081728/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1084757976444837898/image.png)

Untuk memungkinkan frida-client berkomunikasi dengan frida-server di perangkat Android. saya akan memberikan panduan tentang cara menginstal frida server pada perangkat android.

Langkah-langkah untuk menginstal frida server pada perangkat android:

Download dan Install Frida Server dari halaman https://github.com/frida/frida/releases. Pastikan kamu memilih versi Frida server yang cocok dengan arsitektur CPU dari virtual device Genymotion yang telah kamu buat.

![image](https://media.discordapp.net/attachments/740245586095112242/1084758226916102164/image.png)

Gunakan frida-server yang sesuai dengan tipe kernelnya. Untuk mengetahui tipe kernel nya masukan perintah:

```sql
adb shell getprop ro.product.cpu.abi
```
![image](https://media.discordapp.net/attachments/740245586095112242/1084758471678906460/image.png)

Setelah berhasil terunduh, taruh Frida server di direktori ```/data/local/tmp/``` di virtual device kamu. Kamu bisa menggunakan command adb push untuk memindahkan Frida server ke direktori tersebut.

![image](https://media.discordapp.net/attachments/740245586095112242/1084758695889608724/image.png)

```sql
adb push frida-server /data/local/tmp/
```
![image](https://media.discordapp.net/attachments/740245586095112242/1084758931496239104/image.png)

Setelah berhasil diinstall, jalankan Frida server Buka shell adb dan jalankan command berikut untuk menjalankan Frida server:

```sql
adb shell "chmod 755 /data/local/tmp/frida-server"
```
![image](https://media.discordapp.net/attachments/740245586095112242/1084759289102610482/image.png)

Selanjutnya, langkah-langkah konfigurasi sertifikat Burp Suite pada perangkat Genymotion:

Export sertifikat Burp Suite melalui GUI Burp Suite. Buka Burp Suite dan buat sertifikat dengan mengklik tab "Proxy" dan memilih opsi "Options". Pilih tab "Import / Export CA Certificate".

![image](https://media.discordapp.net/attachments/740245586095112242/1084759448846868510/image.png)

Pada bagian "```Export```", pilih opsi "```Certificate in DER Format```". Kemudian klik tombol "```Next```". Selanjutnya, pilih lokasi di mana sertifikat akan disimpan dan simpan dengan nama "```cert-der.crt```".

![image](https://media.discordapp.net/attachments/740245586095112242/1084759864468836372/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1084759991090675752/image.png)

Pertama, kita perlu mengubah sertifikat Burp menjadi format PEM. Gunakan openssl untuk mengonversi DER ke PEM, lalu tampilkan subject_hash_old:

```sql
openssl x509 -inform DER -in Burp_cert.cer -out Burp_cert.pem
openssl x509 -inform PEM -subject_hash_old -in Burp_cert.pem |head -1
```
![image](https://media.discordapp.net/attachments/740245586095112242/1084760253951909898/image.png)

Kemudian, ganti nama file dengan hash keluaran dari perintah terakhir. Misalnya, jika hash adalah 9a5ba575, ganti nama file menjadi 9a5ba575.0:

```sql
mv Burp_cert.pem 9a5ba575.0
```

![image](https://media.discordapp.net/attachments/740245586095112242/1084760442863357952/image.png)

Selanjutnya kita kirim sertifikat Burp Suite ke dalam perangkat Android dengan ADB. Buka Command Prompt atau Terminal pada komputer Anda dan navigasikan ke direktori kerja Anda. Ketik perintah berikut untuk mengirim sertifikat ke perangkat Android Anda melalui adb. Upload dan pasang sertifikat .0:

```bash
# remount the system partition
adb remount
# Upload the certificate
adb push <cert>.0 /system/etc/security/cacerts/
# Change the certificate permissions
adb shell chmod 664 /system/etc/security/cacerts/<cert>.0
```

![image](https://media.discordapp.net/attachments/740245586095112242/1084760703996543097/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1084760837002108969/image.png)

Selanjutnya restart device, Setelah perangkat melakukan boot ulang, cari Pengaturan -> Keamanan -> Kredensial Tepercaya akan menampilkan "```Portswigger CA```" baru sebagai CA tepercaya sistem:

![image](https://media.discordapp.net/attachments/740245586095112242/1084761042372010035/image.png)

Selanjutnya modif proxy Burp suite, Agar Burp Suite dapat mengintersep traffic dari perangkat Android, maka perlu menambahkan alamat IP perangkat Android ke daftar Proxy Listeners pada Burp Suite. Untuk melakukannya, silakan pergi ke Tab Proxy -> Intercept -> Add.

![image](https://media.discordapp.net/attachments/740245586095112242/1084761211465371678/image.png)

Untuk mengubah proxy pada jaringan perangkat Android menggunakan IP Localhost, ikuti langkah berikut:

1.	Buka menu Pengaturan pada perangkat Android
2.	Pilih opsi Wi-Fi
3.	Cari dan pilih SSID yang ingin diubah
4.	Tekan tombol Modifikasi Jaringan
5.	Pada opsi Proxy, pilih Manual
6.	Masukkan IP Localhost dan nomor port yang sesuai
7.	Simpan pengaturan yang telah diubah

![image](https://media.discordapp.net/attachments/740245586095112242/1084761364100304896/image.png)

## Bypass SSL Pinning dengan Frida

Setelah melakukan proses instalasi, terkadang masih ada beberapa langkah tambahan yang perlu dilakukan agar aplikasi atau perangkat dapat berfungsi dengan baik. Namun demikian, setelah proses instalasi selesai dilakukan, masih terdapat kendala yang harus diatasi. Salah satu kendala tersebut adalah ketika traffic belum dapat diintersep karena Frida Server dan Script belum dijalankan pada perangkat atau emulator Android. Oleh karena itu, perlu dilakukan beberapa langkah tambahan untuk menjalankan Frida Server dan Script pada perangkat atau emulator Android agar dapat melakukan intercept pada traffic.

Traffic intercept dapat dilakukan dengan menjalankan Frida server dan menyuntikan script pada target aplikasi (contoh twitter). Untuk mengidentifikasi target aplikasi jalankan aplikasi target (contoh twitter) pada perangkat lalu masukan perintah Frida-ps –U –ai pada terminal/command prompt.
![image](https://media.discordapp.net/attachments/740245586095112242/1084761539145383936/image.png)

Berdasarkan instruksi tersebut, diperoleh PID dan nama proses, dalam contoh ini adalah Mifx dengan PID 4124 dan identifier ```com.mifxid.app``` Setelah mengetahui PID dan nama proses, langkah selanjutnya adalah menjalankan server Frida pada perangkat aplikasi dengan memasukkan perintah yang sesuai.

```bash
adb shell /data/local/tmp/frida-server-16.0.11-android-x86
```
![image](https://media.discordapp.net/attachments/740245586095112242/1084761789507575818/image.png)

Langkah selanjutnya adalah menyuntikkan script SSLpinningBypass.js ke dalam identifier target aplikasi agar sertifikat Burp Suite dapat dipercayai oleh target aplikasi. Hal ini dapat dilakukan dengan menjalankan perintah "```frida -U -f <identifier> -l <file_script> --no-auto-reload```". Jika berhasil, target aplikasi akan secara otomatis dimuat ulang.

```sql
frida -U -f com.mifxid.app -l SSLpinningBypass.js --no-auto-reload
```
![image](https://media.discordapp.net/attachments/740245586095112242/1084762189358973018/image.png)

