---
title: "HackTheBox - SeeTheSharpFlag WriteUps"
date: 2023-02-06 02:00:00 +07:00
categories: [WriteUps]
tags: [ctf, hackthebox, pentest, writeups]
image: 
---

# HackTheBox SeeTheSharpFlag WriteUps

```
Saya telah membuat aplikasi verifikasi kata sandi. Jika saya dapat mengingat kata sandinya, aplikasi akan memberi tahu saya bahwa itu benar. Lihat apakah Anda dapat menebak kata sandi saya.
```

Berikut ini tampilan dibawah bila aplikasi apk tersebut dibuka

![image](https://media.discordapp.net/attachments/740245586095112242/1071993124685103194/image.png)

Setelah itu apa yang bisa lakukan dengan ini, kita mencoba menginputkan random password tetapi aplikasi tersebut menolak dengan tampilan seperti gambar

![image](https://media.discordapp.net/attachments/740245586095112242/1072000486103842878/image.png)

Selanjutnya kita membuka jadx ternyata source code yang digunakan ini menggunakan Xamarin. Kita dapat melihat aplikasi ini dibangun di atas Xamarin , platform sumber terbuka untuk membangun aplikasi modern dan berkinerja baik untuk iOS, Android, dan Windows dengan .NET

![image](https://media.discordapp.net/attachments/740245586095112242/1072000706053152909/image.png)

Setelah itu kita mengcoba decompille source code Apk dengan apktool

![image](https://media.discordapp.net/attachments/740245586095112242/1072021190622846996/image.png)

Dari source code apk tersebut terdapat folder assemblies dan disini mencoba melihat source code dll dari SeeTheSharpFlag.dll

![image](https://media.discordapp.net/attachments/740245586095112242/1072022080750637107/image.png)

Kita membukanya dengan dnSpy dan ternyata file tersebut tidak bisa dibuka secara langsung

![image](https://media.discordapp.net/attachments/740245586095112242/1072023039979573258/image.png)

Mencoba membaca dll file tersebut

![image](https://media.discordapp.net/attachments/740245586095112242/1072034362406735902/image.png)

Untuk mendapatkan file dll yang dapat dibaca di dnspy, kita perlu mengekstrak dan membongkar panjang payload untuk mengetahui panjang payload dan kemudian LZ4 mendekompresi payload tersebut.

Kita mencoba menggunakan decompiler dari python yang didapat dari github [Xamarin Decompres](https://github.com/NickstaDB/xamarin-decompress) 

![image](https://media.discordapp.net/attachments/740245586095112242/1072037490329923665/image.png)

Buka lagi dengan dnSpy dan kita mendapatkan source code dari dll tersebut dan coba ada hal apa yang menarik disini

![image](https://media.discordapp.net/attachments/740245586095112242/1072037767162376312/image.png)

Setelah mencari cari kita mendapatkan suatu modul button yang dimana ini mengarah pada click tombol pada Aplikasi

![image](https://media.discordapp.net/attachments/740245586095112242/1072038024336134204/image.png)

Dan Kita bisa melihat perbandingan string di sini, ini adalah nilai rahasia di aplikasi dan masukan kita

![image](https://media.discordapp.net/attachments/740245586095112242/1072038244243488778/image.png)

Mencoba memodifikasi dengan dnSpy

![image](https://media.discordapp.net/attachments/740245586095112242/1072038480571547719/image.png)

Kita bisa lihat setelah pemanggilan ``System.String::op_Equality`` , instruksi brfalse.s akan melompat ke ``ldarg.0`` , yang merupakan rute sukses

![image](https://media.discordapp.net/attachments/740245586095112242/1072038750131068968/image.png)

Disini setelah kita ganti dengan value seperti gambar dibawah selanjutnya kita save module tersebut

![image](https://media.discordapp.net/attachments/740245586095112242/1072039044340531291/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1072039239723782244/image.png)

Build module nya dengan apktool

![image](https://media.discordapp.net/attachments/740245586095112242/1072039438013693962/image.png)

Verifikasi signer menggunakan jarsigner

![image](https://media.discordapp.net/attachments/740245586095112242/1072039591709790238/image.png)

Mengganti nama apk dengan zipalign

![image](https://media.discordapp.net/attachments/740245586095112242/1072039835969261628/image.png)

Setelah itu kita mencoba membuka aplikasi tersebut. disini bila kita menginputkan ``1234`` atau apapun input itu kita mendapatkan flag seperti gambar dibawah dan disini berhasil kita mengganti source code yang ada pada dll

![image](https://media.discordapp.net/attachments/740245586095112242/1072040128949788732/image.png)

Tetapi disini dimana letak Flag HTB nya? Karena kita hanya merubah route dari fail menjadi success, flag disimpan di variabel streamReader yang tidak tercetak. Idenya adalah kita akan mengubah ``streamReader.ReadToEnd()`` menjadi array acak.ReadToEnd() dalam perbandingan karena ``ReadToEnd`` akan membaca semua karakter dari posisi saat ini hingga akhir aliran, jika kita membaca dalam perbandingan maka kita tidak dapat membaca lagi.

Kemudian kami menempatkan ``streamReader.ReadToEnd()`` ke dalam pesan Output di rute sukses, dan terakhir memodifikasi aliran untuk selalu datang ke rute sukses jika salah memasukkan rahasia/kata sandi.


Selanjutnya kita mencoba mengganti valuenya seperti digambar 

![image](https://media.discordapp.net/attachments/740245586095112242/1072040622279634954/image.png)

Menjadi

![image](https://media.discordapp.net/attachments/740245586095112242/1072040781902262322/image.png)

Mencoba mengubahnya lagi

![image](https://media.discordapp.net/attachments/740245586095112242/1072041034260946964/image.png)

Menjadi

![image](https://media.discordapp.net/attachments/740245586095112242/1072041266851876884/image.png)

Setelah selesai Simpan ke modul baru, timpa dll di folder dekompilasi, repacking, sign, zipalign, install. Dan bila kita menginputkan random value lagi di aplikasi kita akan mendapatkan Flagnya

![image](https://media.discordapp.net/attachments/740245586095112242/1072044658278006794/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1072041535358631986/image.png?width=645&height=606)
