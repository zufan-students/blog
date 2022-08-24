---
title: "All Defense Tool"
date: 2022-08-04 04:00:00 +07:00
categories: [Tools]
tags: [APT, ASSETDISCOVERY, ATTACKSURFACE, BOUNTYHUNTERS, BUGBOUNTY, DEFENSE, EXPLOIT, HACKING, LATERALMOVEMENT, OSINT, PENETRATIONTESTING, SCANNER, SCANNER AND EXPLOITER, THREATHUNTING, VAPT, VULNERABILITY]
image: 
---

# All Defense Tool
​ First of all, congratulations on finding the treasure. This project integrates excellent open source offensive and defensive weapons projects on the entire network, including information collection tools (automated utilization tools, asset discovery tools, directory scanning tools, subdomain name collection tools, fingerprint identification tools, port scanning tools, various plug-ins... etc...), Vulnerability Exploitation Tools (Major CMS Exploitation Tools, Middleware Exploitation Tools, etc...), Intranet Penetration Tools (Tunnel Agent, Password Extraction...), Emergency Response tools, Party A's operation and maintenance tools, and other security offensive and defensive data are organized for use by both offensive and defensive parties. If you have better suggestions, pull requests are welcome.

## Disclaimer

**Key reminder: The tools of this project come from the Internet. Please identify whether it contains Trojan horses and backdoors! ! Hvv is coming soon, please be vigilant! ! ! **

1. `All contents of this project are for study and research purposes only. Please do not use the technical means of the project for illegal purposes. Any negative impact caused by anyone has nothing to do with me.`
2. `All content and news in this document do not represent my attitude or position. If you have suggestions or plans, please submit issues`
3. `No advertising fees will be charged, and all tool links displayed have nothing to do with me`

# Table of contents

* [Semi/Fully Automatic Utilization Tool](#semi-fully automated utilization tool)
* [Information Collection Tool](#Information Collection Tool)
  * [Asset Discovery Tool](#Asset Discovery Tool)
  * [Subdomain collection tool](#Subdomain collection tool)
  * [Directory Scanning Tool](#Directory Scanning Tool)
  * [Fingerprint identification tool](#fingerprint identification tool)
  * [Port Scanning Tool](#Port Scanning Tool)
  * [Burp plugin](#burp plugin)
  * [Browser plugin](#Browser plugin)
  * [Mailbox &amp; Phishing](#mailboxphishing)
  * [Social Worker Personal Information Collection] (#Social Worker Personal Information Collection)
  * [Commonly used gadgets](#commonly used gadgets)
* [Exploit Tool](#ExploitTool)
  * [Vulnerability scanning framework/tool](#vulnerability scanning framework tool)
  * [Middleware exploit tool](#Middleware exploit tool)
  * [Key cms utilization tool] (#Key cms utilization tool)
  * [Information Leak Utilization Tool](#Information Leak Utilization Tool)
  * [Database Utilization Tool](#Database Utilization Tool)
  * [Blasting tool](#blasting tool)
  * [Whole network dictionary collection](#Whole network dictionary collection)
  * [regular exploit tool](#regular exploit tool)
  * [Deserialization Utilization Tool](#Deserialization Utilization Tool)
  * [Code Audit Auxiliary Tool](#Code Audit Auxiliary Tool)
* [Intranet penetration tool](#Intranet penetration tool)
  * [Privilege Escalation Project](#Privilege Escalation Project)
  * [Transverse tool](#Transverse tool)
  * [shell hosting tool](#shell hosting tool)
  * [password extraction tool](#password extraction tool)
  * [Tunnel Proxy Tool](#Tunnel Proxy Tool)
  * [Excellent kill-free project] (#Excellent kill-free project)
  * [Permission maintenance tool](#Permission maintenance tool)
* [Operation &amp; Party A &amp; Defender Tools] (#Operation and Maintenance Party A Defender Tools)
  * [Linux Emergency Response Tool](#linux Emergency Response Tool)
  * [Windows Emergency Response Tool](#windows Emergency Response Tool)
  * [Memory Mac killing tool](#Memory Mac killing tool)
  * [xxxx](#xxxx)
  * [Trace counter tool](#Trace counter tool)
* [Security Data Arrangement](#Security Data Arrangement)
  * [Red and Blue Data Collection] (#Red and Blue Data Collection)
  * [Cloud Security Information](#Cloud Security Information)
  * [Range List](#Range List)
  * [Infrastructure and Environment Construction] (#Infrastructure and Environment Construction)

Warm reminder: Don't indulge in offense and defense and forget to eat~

- Programmer's guide on how to cook at home. https://github.com/Anduin2017/HowToCook

# Semi/Fully Automated Exploitation Tool

| Project Introduction | Project Address | Project Name |
| ------------------------------------------------- ----------- | -------------------------------------- -------- | ------------- |
| One-stop service, you only need to enter the root domain name to collect relevant assets in all directions and detect vulnerabilities. You can also enter multiple domain names, C-segment IP, etc., see below for specific cases. | https://github.com/0x727/ShuiZe_0x727 | ShuiZe_0x727 |
| Individual combat arsenal, you deserve it | https://github.com/yaklang/yakit | yakit |
| Automated cruise scanning framework (available for red team evaluation) | https://github.com/b0bac/ApolloScanner | ApolloScanner |
| Automatic port scanning, TCP fingerprinting and banner capture for specified IP segments, asset lists, and surviving network segments | https://github.com/lcvvvv/kscan | kscan |
| An Unexplored Vulnerability Scanning Tool | https://github.com/broken5/bscan | bscan |
| A vulnerability scanner glue, 30 tools are automatically invoked after adding a target | https://github.com/78778443/QingScan | QingScan |
| Distributed Asset Information Collection and Vulnerability Scanning Platform | https://github.com/1in9e/gosint | gosint |
| A comprehensive tool to assist common penetration testing projects or quick management of offensive and defensive projects | https://github.com/P1-Team/AlliN | AlliN |
| nemo_go automated information collection | https://github.com/hanc00l/nemo_go | nemo_go |
| Integrated asset management system from subdomains, port services, vulnerabilities, crawlers, etc. | https://github.com/CTF-MissFeng/bayonet | bayonet |
| A highly customizable web automated scanning framework | https://github.com/r3curs1v3-pr0xy/vajra | vajra |
| reconFTW is an information collection tool that integrates 30 tools | https://github.com/six2dez/reconftw | reconftw |
| Automated Detection Framework | https://github.com/yogeshojha/rengine | rengine |
| GUI interface automation tool | https://github.com/lz520520/railgun | Railgun |
| Online cms identification\|information leakage\|industrial control\|system\|Internet of things security\|cms vulnerability scan\|nmap port scan\|subdomain acquisition\|to be continued.. | https://github.com/iceyhexman /onlinetools | Online toolset |
| Acunetix Web Vulnerability Scanner GUI Version] | https://github.com/x364e3ab6/AWVS-13-SCAN-PLUS | AWVS-GUI |

# Information collection tools

## Asset Discovery Tool

| Project Introduction | Project Address | Project Name |
| ------------------------------------------------- ----------- | -------------------------------------- ------ | --------------- |
| reconFTW is an information collection tool that integrates 30 tools | https://github.com/six2dez/reconftw | reconftw |
| Asset Infinite Cruise Scanning System | https://github.com/awake1t/linglong | linglong |
| SRC subdomain asset monitoring | https://github.com/LangziFun/LangSrcCurise | LangSrcCurise |
| Quickly scout Internet assets associated with targets and build a basic asset information base. | https://github.com/TophantTechnology/ARL | ARL (Lighthouse) |
| Mobile terminal (Android, iOS, WEB, H5, static website) information collection scanning tool | https://github.com/kelvinBen/AppInfoScanner | AppInfoScanner |
| Integrate GoogleHacking syntax for information collection | https://github.com/TebbaaX/GRecon | Grecon |
| Fetch landing page content from third-party platforms | https://github.com/tomnomnom/waybackurls | waybackurls |
| Extract target-related information from multiple websites | https://github.com/lc/gau | gau |
| A collection of multiple network mapping platforms, you can quickly search for information on multiple network mapping platforms and combine display and export. | https://github.com/ExpLangcn/InfoSearchAll | InfoSearchAll |
| Call ZoomEye's official api---GUI interface
