# TryHack3M: Bricks Heist
<img width="1200" height="1200" alt="Pasted image 20260407110348" src="https://github.com/user-attachments/assets/39ee46b9-8303-4466-a563-c51414ea2bc9" />

רמה: קלה (מצויין למתחילים/ חימום)
קטגוריה: (web)(crypto)
מטרה: `10.114.136.246`


דבר ראשון שאני עושה בכל חדר או CTF שאני מתחיל בלבדוק את החיבור שלי לשרת המטרה ולרשת האתגר דרך הVPN בעזרת ping .
לאחר שבדקנו את החיבור והכל תקין אנחנו יכולים להמשיך לשלב ה- recon

דבר שני, האתגר ביקש לפני שמתחילים להשים את הדומיין `bricks.thm`
עם כתובת הIP של המטרה בתוך התיקיה `/etc/hosts`

<img width="1501" height="787" alt="צילום מסך 2026-04-07 113623" src="https://github.com/user-attachments/assets/a5e20f39-9d1c-4d57-800c-e993b7082ecf" />

אחרי שסיימנו אם כל זה אנחנו יכולים להתחיל את תהליך הפיתרון
### שלב 1: איסוף מידע (recon)

בשביל לקבל מידע של איזה שירותים רצים על השרת הזה אני משתמש בכלי `nmap` כלי מצוין למציאת גרסאות, שירותים, ודרכי גישה לשרת.
הפקודה שהשתמשתי בה: `nmap -sC -sV 10.114.136.246 -F`
תוצאת הפלט של הכלי:

``` nse
Starting Nmap 7.95 ( https://nmap.org ) at 2026-04-07 11:06 IDT
Stats: 0:00:13 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 75.00% done; ETC: 11:06 (0:00:04 remaining)
Nmap scan report for 10.114.136.246
Host is up (0.078s latency).
Not shown: 96 closed tcp ports (reset)
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 67:6d:d9:4c:a1:8e:34:07:71:1c:71:d7:e4:47:f7:de (RSA)
|   256 d4:7d:00:b6:25:91:55:fe:9e:76:87:b8:14:3c:17:ab (ECDSA)
|_  256 e2:d3:2a:b2:48:dd:74:cb:65:2c:16:f0:32:66:97:f5 (ED25519)
80/tcp   open  http     Python http.server 3.5 - 3.10
|_http-server-header: WebSockify Python/3.8.10
|_http-title: Error response
443/tcp  open  ssl/http Apache httpd
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: organizationName=Internet Widgits Pty Ltd/stateOrProvinceName=Some-State/countryName=US
| Not valid before: 2024-04-02T11:59:14
|_Not valid after:  2025-04-02T11:59:14
|_http-server-header: Apache
| tls-alpn:
|   h2
|_  http/1.1
| http-robots.txt: 1 disallowed entry
|_/wp-admin/
|_http-generator: WordPress 6.5
|_http-title: Brick by Brick
3306/tcp open  mysql    MySQL (unauthorized)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 29.28 seconds

```

אנחנו רואים כמה דברים מעניינים:
1. פורט 22 SSH לחיבור מרוחק אך בצורה מאובטחת
2. פורט 80 HTTP שעובד על python ולאחר בדיקה מסתבר שהוא מחזיר קוד 405 ואין שם תוכן או קבצים
3. פורט 443 HTTPS שעובד על wordpress והאתר כן פעיל 
4. פורט 3306 mysql כנראה הבסיס נתונים של האתר

והפעלתי גם את הכלי nuclei כדי לחפש אם יש איזה פגיעות כלשהי באתר בשביל לנצל אותה ולקבל גישה:
``` nse 
[INF] Current nuclei version: v3.7.1 (latest)
[INF] Current nuclei-templates version: v10.4.1 (latest)
[INF] New templates added in latest release: 76
[INF] Templates loaded for current scan: 9976
[INF] Executing 9959 signed templates from projectdiscovery/nuclei-templates
[WRN] Loading 17 unsigned templates for scan. Use with caution.
[INF] Targets loaded for current scan: 1
[INF] Templates clustered: 2260 (Reduced 2134 Requests)
[INF] Using Interactsh Server: oast.fun
[CVE-2024-2473] [http] [medium] https://bricks.thm/wp-admin/?action=postpass
[CVE-2024-25600] [http] [critical] https://bricks.thm/wp-json/bricks/v1/render_element
```

### שלב 2: ניצול הפגיעות והשגת המטרות

אנחנו רואים שיש לנו פגיעות קריטית של `CVE-2024-25600` ולאחר חיפוש בגיטהאב אנחנו רואים את הדף הבא:

<img width="1866" height="746" alt="צילום מסך 2026-04-07 112322" src="https://github.com/user-attachments/assets/1db6b005-4efd-443d-9d85-82913a8f3ed8" />

הורדתי את התיקייה מהגיט והשתמשתי בכלי על מנת לקבל גישה לשרת הפגיע.
<img width="1497" height="889" alt="צילום מסך 2026-04-07 114628" src="https://github.com/user-attachments/assets/2d5d1824-75f1-4633-9974-b3471eab4f79" />

ואנחנו רואים שקיבלנו גישה למערכת ויש לנו קובץ סודי בתוך התיקייה של האתר, כשפותחים אותו מגלים דגל ואנחנו יכולים לענות על השאלה הראשונה
#### What is the content of the hidden .txt file in the web folder?
``` 
THM{fl46_650c844110baced87e1606453b93f22a}
```

אחרי זה שואלים אותנו `מה השם של התהליך החשוד במערכת`
ישר השתמשתי ב `systemct`, ליתר דיוק בפקודה הספציפית הזאת שמראה איזה שירותים רצים באותו הרגע: 
``` unix
systemctl list-units --type=service --state=running
```

אחרי זה אנחנו רואים את הדבר הבא:

<img width="1086" height="217" alt="צילום מסך 2026-04-07 115544" src="https://github.com/user-attachments/assets/c5284fe8-ce3d-477d-9d80-5c2f8c68615d" />

התהליך של ubuntu.service נראה כמו התהליך החשוד שרץ
בגלל שהוא מתואר כ- `tryhack3m` ולשם כך אני רוצה לראות את תוכן התהליך אז השתמשתי בפקודה הבאה וזה מה שקיבלתי:

``` nse 
apache@ip-10-114-136-246:/data/www/default$ systemctl cat ubuntu.service
systemctl cat ubuntu.service
# /etc/systemd/system/ubuntu.service
[Unit]
Description=TRYHACK3M

[Service]
Type=simple
ExecStart=/lib/NetworkManager/nm-inet-dialog
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

גילינו את השם שלו ואנחנו יכולים לענות על השתי שאלות:
#### What is the name of the suspicious process?
```
nm-inet-dialog
```

#### What is the service name affiliated with the suspicious process?
```
ubuntu.service
```

לאחר מכן השאלה הבאה היא: `מה הוא השם של הקובץ לוג של הכורה`
כורה הכוונה ל `crypto mining` והם מבקשים את השם של הקובץ לוג
אנחנו רואים את התיקייה שבה התהליך נמצא ואנחנו יכולים לבדוק מהלוגים בתיקייה איזה אחד הוא של הכרייה
בתוך התיקייה מצאתי את הקובץ `inet.conf` וכשפתחתי אותה לראות את התוכן גיליתי את הדבר הבא:

<img width="1352" height="349" alt="צילום מסך 2026-04-07 120336" src="https://github.com/user-attachments/assets/08688910-6f12-4b7b-b240-0828f0fc7a70" />

אז קודם כל נרשום את השם של הקובץ בשאלה ונמשיך לשאלה הבאה.

#### What is the log file name of the miner instance?
```
inet.conf
```

אנחנו רואים שגילינו איזה ID מסויים בתוך הלוג:
```
5757314e65474e5962484a4f656d787457544e424e574648555446684d3070735930684b616c70555a7a566b52335276546b686b65575248647a525a57466f77546b64334d6b347a526d685a6255313459316873636b35366247315a4d304531595564476130355864486c6157454a3557544a564e453959556e4a685246497a5932355363303948526a4a6b52464a7a546d706b65466c525054303d
```

השתמשתי ב- `CyberChef` כדי לגלות מה יש מאחורי ההצפנה וזה מה שגיליתי:
<img width="1536" height="781" alt="Pasted image 20260407140818" src="https://github.com/user-attachments/assets/5ccae011-baf3-42fe-b989-97984bae1580" />

קיבלתי את הרצף הבא:
```
bc1qyk79fcp9hd5kreprce89tkh4wrtl8avt4l67qabc1qyk79fcp9had5kreprce89tkh4wrtl8avt4l67qa
```

שכשבודקים בגוגל מה זאת ההצפנה הזאת מגלים את הדבר הבא:
<img width="1350" height="689" alt="צילום מסך 2026-04-07 120556" src="https://github.com/user-attachments/assets/4832aa28-251f-47ef-94fa-4141f9963b9f" />

מסתבר שזה 2 כתובות לביטקוין:
```
bc1qyk79fcp9hd5kreprce89tkh4wrtl8avt4l67qa
bc1qyk79fcp9had5kreprce89tkh4wrtl8avt4l67qa
```

בדקתי באתר `https://www.blockchain.com/explorer/` כל כתובת בנפרד והכתובת הראשונה היחידה שעבדה והיא הביאה לי את הארנק של הכורה:

<img width="1463" height="777" alt="צילום מסך 2026-04-07 141420" src="https://github.com/user-attachments/assets/b959848e-9e08-4b4c-828d-39431c11bd2a" />

#### What is the wallet address of the miner instance?
```
bc1qyk79fcp9hd5kreprce89tkh4wrtl8avt4l67qa
```

והשאלה האחרונה ששואלים אותנו `איזה מהארנקים בהסטוריית העברה, שייח לקבוצת הונאה ותמצא למי הוא שייך`. נכנסתי להיסטוריה ובדקתי אחד אחד עד שמצאתי את זה:

<img width="1090" height="334" alt="צילום מסך 2026-04-07 141436" src="https://github.com/user-attachments/assets/af138111-d509-4cb8-af05-f3a4e0fc638b" />

כשחיפשתי בגוגל את הכתובת ארנק: `32pTjxTNi7snk8sodrgfmdKao3DEn1nVJM` וזה שייך לקבוצה מסוכנת בשם LockBit

<img width="1318" height="614" alt="צילום מסך 2026-04-07 121128" src="https://github.com/user-attachments/assets/59a48275-480e-494b-8e6a-a68efe8a3ff1" />

#### The wallet address used has been involved in transactions between wallets belonging to which threat group?
```
LockBit
```

וככה סיימנו את האתגר הזה.

<img width="1672" height="897" alt="Pasted image 20260407142946" src="https://github.com/user-attachments/assets/e9c2aeb8-11e5-49b4-a5e3-f300831dfbff" />

