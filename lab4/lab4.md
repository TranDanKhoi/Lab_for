# Lab 4 Forensic

Link bài: https://github.com/vonderchild/digital-forensics-lab/tree/main/Lab%2004

## You are a cyber security specialist who has been called upon to investigate a major cyber security breach. The company's web server has been compromised, and the attacker has attempted to exploit multiple vulnerabilities. You've been given the task of piecing together the attacker's intentions and uncovering the extent of the damage. With that in mind, your challenge is to answer the following questions:

### 1. What IP address does the attack seem to be originating from?
Em và đọc file `.log` thì thấy chủ yếu là IP **192.168.0.106** và cũng đã có bằng chứng xác thực ở câu 2

### 2. Which vulnerabilities do you think are being exploited, and what evidence do you have to support your findings?

Dựa vào những gì lab 4 dạy thì có những lỗ hổng sau: **Path Traversal**, **Remote Command Execution** và **SQL Injection**

Bằng chứng:
* Path Traversal

```bash
192.168.0.106 - - [16/Feb/2023:01:35:07 +0500] "GET /images.php?file=%2Fetc%2Fpasswd HTTP/1.1" 302 2422 "http://192.168.0.101:9090/images.php" "Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0"
192.168.0.106 - - [16/Feb/2023:01:35:07 +0500] "GET /view.php?image=/etc/passwd HTTP/1.1" 200 202 "http://192.168.0.101:9090/images.php" "Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0"
192.168.0.106 - - [16/Feb/2023:01:35:13 +0500] "GET /view.php?image=../etc/passwd HTTP/1.1" 200 203 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0"
...
cat modsec_audit.log | grep 'REQUEST_URI_RAW'
Message: Warning. Pattern match "(?i)(?:\\x5c|(?:%(?:c(?:0%(?:[2aq]f|5c|9v)|1%(?:[19p]c|8s|af))|2(?:5(?:c(?:0%25af|1%259c)|2f|5c)|%46|f)|(?:(?:f(?:8%8)?0%8|e)0%80%a|bg%q)f|%3(?:2(?:%(?:%6|4)6|F)|5%%63)|u(?:221[56]|002f|EFC8|F025)|1u|5c)|0x(?:2f|5c)|\\/))(?:%(?:(?:f(?:(?:c%80|8)%8)?0%8 ..." at REQUEST_URI_RAW. [file "/usr/share/modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf"] [line "47"] [id "930100"] [msg "Path Traversal Attack (/../)"] [data "Matched Data: /../ found within REQUEST_URI_RAW: /view.php?image=../../etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.2"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-lfi"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/255/153/126"]
```

* Remote Command Execution

```bash
cat error.log | grep 'ARGS:cmd'
[Thu Feb 16 01:35:45.748064 2023] [:error] [pid 274] [client 192.168.0.106:45862] [client 192.168.0.106] ModSecurity: Warning. Matched phrase "etc/shadow" at ARGS:cmd. [file "/usr/share/modsecurity-crs/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf"] [line "97"] [id "930120"] [msg "OS File Access Attempt"] [data "Matched Data: etc/shadow found within ARGS:cmd: cat /etc/shadow"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.2"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-lfi"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/255/153/126"] [tag "PCI/6.5.4"] [hostname "192.168.0.101"] [uri "/command.php"] [unique_id "Y-1CIdQo6pgKlS9WQB6nLgAAAAQ"], referer: http://192.168.0.101:9090/command.php #LFI
[Thu Feb 16 01:35:07.440596 2023] [:error] [pid 271] [client 192.168.0.106:54690] [client 192.168.0.106] ModSecurity: Warning. Matched phrase "etc/passwd" at ARGS:file. [file "/usr/share/modsecurity-crs/rules/REQUEST-932-APPLICATION-ATTACK-RCE.conf"] [line "500"] [id "932160"] [msg "Remote Command Execution: Unix Shell Code Found"] [data "Matched Data: etc/passwd found within ARGS:file: /etc/passwd"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.2"] [tag "application-multi"] [tag "language-shell"] [tag "platform-unix"] [tag "attack-rce"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/152/248/88"] [tag "PCI/6.5.2"] [hostname "192.168.0.101"] [uri "/images.php"] [unique_id "Y-1B-x1ILvAGFIiKbK7QJQAAAAE"], referer: http://192.168.0.101:9090/images.php #RCE
...
```

* SQLi

```bash
cat error.log | grep 'SQLi'            
[Thu Feb 16 01:36:25.133354 2023] [:error] [pid 271] [client 192.168.0.106:46382] [client 192.168.0.106] ModSecurity: Warning. detected SQLi using libinjection with fingerprint 's&1c' [file "/usr/share/modsecurity-crs/rules/REQUEST-942-APPLICATION-ATTACK-SQLI.conf"] [line "65"] [id "942100"] [msg "SQL Injection Attack Detected via libinjection"] [data "Matched Data: s&1c found within ARGS:search: user2' and 1=1#"] [severity "CRITICAL"] [ver "OWASP_CRS/3.3.2"] [tag "application-multi"] [tag "language-multi"] [tag "platform-multi"] [tag "attack-sqli"] [tag "paranoia-level/1"] [tag "OWASP_CRS"] [tag "capec/1000/152/248/66"] [tag "PCI/6.5.2"] [hostname "192.168.0.101"] [uri "/users.php"] [unique_id "Y-1CSR1ILvAGFIiKbK7QJwAAAAE"], referer: http://192.168.0.101:9090/users.php
...
```

### 3. How can we determine what web browser the attacker is using?
Firefox do em thấy `access.log` dòng nào cũng có

### 4. Did the attacker use any automated tools during the attack? If so, can you identify the name of the tool and its purpose?

Em nghĩ là hacker có sài tool và em tìm được tool `SQLmap` qua `access.log`

```bash
192.168.0.106 - - [16/Feb/2023:01:38:53 +0500] "GET /users.php HTTP/1.1" 200 1117 "-" "sqlmap/1.6.11#stable (https://sqlmap.org)"
192.168.0.106 - - [16/Feb/2023:01:38:53 +0500] "GET /users.php HTTP/1.1" 200 1117 "-" "sqlmap/1.6.11#stable (https://sqlmap.org)"
192.168.0.106 - - [16/Feb/2023:01:39:00 +0500] "POST /users.php HTTP/1.1" 200 1050 "-" "sqlmap/1.6.11#stable (https://sqlmap.org)"
```

Còn mấy lỗ hổng khác thì em không thể xác định chính xác được tool gì có thể là tự code hack được

### 5. Which file was the attacker trying to access but couldn't due to limited server access?

Em đọc `access.log` thấy có `../../../../etc/shadow` truy cập có 1 lần nên em thử qua bên `error.log` đọc thử

```bash
cat error.log | grep '../../../../etc/shadow'
[Thu Feb 16 01:35:30.820208 2023] [php:warn] [pid 273] [client 192.168.0.106:60308] PHP Warning:  file_get_contents(images/../../../../etc/shadow): Failed to open stream: Permission denied in /var/www/html/view.php on line 10
```

### 6. Did the attacker gain access to any confidential data? If yes, how much data was compromised?

`etc/passwd`: 20 users, 650 bytes

```bash
192.168.0.106 - - [16/Feb/2023:01:35:27 +0500] "GET /view.php?image=../../../../etc/passwd HTTP/1.1" 200 650 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0" #bytes

--f0015c41-E-- #users
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
mysql:x:101:101:MySQL Server,,,:/nonexistent:/bin/false
```

`database.sql`: 5 users + password 620 bytes
`important_note.txt`: 345 bytes

```bash
-rw-r--r--   1 root root  620 Feb 12 15:28 database.sql
-rw-r--r--   1 root root  345 Feb 16 01:34 important_note.txt
```

### 7. An important secret was compromised. Can you figure it out? Hint: The secret you're looking for is not in a `.sql` or a `.php` file

Theo hint thì chỉ còn `important_note.txt` và nội dung bên trong 

```bash
Hey there! Just a heads up - if we don't add security checks to our web app, our top-secret files might as well be written on a billboard. And trust me, we don't want that kind of attention. So let's get those checks in place, okay? We wouldn't want the world to know that our password is 'sup3r_s3cr3t_4nd_1mp0rt4nt_p4ssw0rd', now would we? ;)
```

### 8. The attacker left a message for the server administrator. Find out what the message said, and also mention how you were able to find it.

Em tìm thấy đoạn sau:

```bash
search=%27+union+select+%27Gentlemen%2C+it+is+with+great+pleasure+I+inform+you+that%3A%27%2C+concat%28char%2890%29%2C+char%28109%29%2C+char%28120%29%2C+char%28104%29%2C+char%2890%29%2C+char%2851%29%2C+char%28116%29%2C+char%28111%29%2C+char%2877%29%2C+char%2888%29%2C+char%2866%29%2C+char%28119%29%2C+char%2877%29%2C+char%2888%29%2C+char%2882%29%2C+char%2853%29%2C+char%2888%29%2C+char%2850%29%2C+char%28103%29%2C+char%28119%29%2C+char%2899%29%2C+char%2872%29%2C+char%2865%29%2C+char%28120%29%2C+char%28100%29%2C+char%2872%29%2C+char%28108%29%2C+char%28102%29%2C+char%28101%29%2C+char%2884%29%2C+char%2866%29%2C+char%2849%29%2C+char%2899%29%2C+char%28108%29%2C+char%2857%29%2C+char%2851%29%2C+char%2877%29%2C+char%2850%29%2C+char%2874%29%2C+char%28122%29%2C+char%2877%29%2C+char%2888%29%2C+char%2881%29%2C+char%28122%29%2C+char%2888%29%2C+char%28122%29%2C+char%2870%29%2C+char%28122%29%2C+char%2888%29%2C+char%2850%29%2C+char%2852%29%2C+char%28119%29%2C+char%28100%29%2C+char%2849%29%2C+char%2857%29%2C+char%28116%29%2C+char%28101%29%2C+char%2886%29%2C+char%2857%29%2C+char%28119%29%2C+char%2899%29%2C+char%28106%29%2C+char%2866%29%2C+char%28119%29%2C+char%2877%29%2C+char%2851%29%2C+char%2874%29%2C+char%2848%29%2C+char%28101%29%2C+char%2883%29%2C+char%2870%29%2C+char%2857%29%29%2C+%27%3AD%27+%23
```

Sau đó em dùng keyword là **Gentlemen** lọc cho nhanh vì em nghĩ sẽ có code web hiện mã

```bash
<td>Gentlemen, it is with great pleasure I inform you that:</td>
                          <td>ZmxhZ3toMXBwMXR5X2gwcHAxdHlfeTB1cl93M2JzMXQzXzFzX24wd19teV9wcjBwM3J0eSF9</td>
```

Dịch ra: **flag{h1pp1ty_h0pp1ty_y0ur_w3bs1t3_1s_n0w_my_pr0p3rty!}**

### 9. What were some indicators that confirmed that an attack had taken place? What were your key takeaways from this attack?

Những dấu hiệu hacker hack vào web:
* Path Travesal: `../../../etc/passwd` như bài ví dụ
* RCE: Trong file `error.log` thường có keyword như `Remote Command Execution`
* SQLi: file `error.log` hay `modsec_audit.log` thường để attack-sqli

Những điều em học được qua lab này:
* Hacker hack vào những lỗ hổng web qua 3 phương pháp phổ biến trên
* Cách nhận biết hacker đã hack web bằng lỗ hổng nào

### 10. Based on this attack, what indicators of compromise can be used to detect future attacks?

Những dấu hiện của 3 lỗ hổng được học trên