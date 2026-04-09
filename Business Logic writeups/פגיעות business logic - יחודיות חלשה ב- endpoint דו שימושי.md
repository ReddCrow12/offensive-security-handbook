# Weak Isolation on Dual-Use Endpoint
**PortSwigger Web Security Academy — Business Logic Vulnerabilities**

---

## קצת הקשר לפני שמתחילים

הלאב הזה הוא אחד מהמועדפים עליי בסדרה של Business Logic, כי הוא ממחיש בצורה מאוד קלינית את ההבדל בין פגיעות טכנית לפגיעות לוגית.

לא צריך פה SQL Injection, לא XSS, לא שום payload מפוצץ. רק להסתכל על בקשת HTTP ולשאול — **מה קורה אם אני פשוט מוציא פרמטר אחד?**

המטרה: להיכנס לחשבון של `administrator` ולמחוק את המשתמש `carlos`.  
הפרטים שיש לנו: `wiener:peter`

---

## נכנסים ומסתכלים מסביב

נכנסים עם wiener, נכנסים לדף החשבון, ורואים טופס שינוי סיסמה רגיל לכאורה.

<img width="1468" height="757" alt="צילום מסך 2026-04-09 131825" src="https://github.com/user-attachments/assets/5bb87dda-2c77-4bb7-a3a6-a96b2f3ea061" />


מיירטים את הבקשה עם Burp כשמשנים סיסמה:

<img width="1873" height="832" alt="צילום מסך 2026-04-09 131959" src="https://github.com/user-attachments/assets/094e9d28-2ff3-4746-91ee-9f79435f73ad" />

```http
POST /my-account/change-password HTTP/2

csrf=g9YVjGyY5qPKpgKrac8df4CqrgBeBtEi
&username=wiener
&current-password=peter
&new-password-1=wiener
&new-password-2=wiener
```

ושם רואים משהו שמיד צריך לצוץ לכם דגל אדום — **`username` בא מהלקוח**.

רגע, למה? השרת יודע מי מחובר דרך ה-session cookie. אין סיבה לשאול את הלקוח מי הוא. זה ריח של עצלות בפיתוח, וריח של עצלות בפיתוח זה ריח של פגיעות.

---

## בודקים את הגבולות

**ניסיון ראשון** — שולחים עם `username=carlos` ועם הסיסמה הנוכחית שלנו (`peter`):

<img width="1619" height="862" alt="צילום מסך 2026-04-09 132234" src="https://github.com/user-attachments/assets/f3294386-d9b7-4e88-83e0-96c34ececc6a" />

נכשל. הגיוני — השרת בדק שהסיסמה הנוכחית תואמת לחשבון `carlos`, ולנו אין אותה.

**ניסיון שני** — שולחים עם `current-password=` ריק לגמרי:

<img width="1539" height="877" alt="צילום מסך 2026-04-09 132248" src="https://github.com/user-attachments/assets/18ea2106-5b28-42ca-b88f-e0103d56d815" />

גם לא עבד. השרת ראה שדה ריק ודחה.

**ניסיון שלישי** — מה אם פשוט **מוחקים את הפרמטר כולו** מהבקשה?

```http
csrf=...&username=carlos&new-password-1=hacked&new-password-2=hacked
```

<img width="1881" height="901" alt="צילום מסך 2026-04-09 132604" src="https://github.com/user-attachments/assets/bfb64c7f-6d79-4919-bc17-a2583e0f0f6a" />

עבד.

---

## מה בעצם קרה פה

הקוד מאחורי הקלעים כנראה נראה בערך ככה:

```php
if (isset($_POST['current-password'])) {
    // תבדוק שהסיסמה נכונה
    verifyPassword($_POST['username'], $_POST['current-password']);
}
// תשנה סיסמה
changePassword($_POST['username'], $_POST['new-password-1']);
```

הבדיקה של הסיסמה הנוכחית עטופה ב-`if` שמסתכל אם **הפרמטר קיים**. אם הפרמטר לא קיים בכלל — הבדיקה פשוט לא מתרחשת, ומשנים סיסמה ישירות.

זו הנחה לוגית שגויה קלאסית: המפתח הניח שהפרמטר תמיד יהיה שם כי הפורם תמיד ישלח אותו. אבל אנחנו לא מוגבלים לפורם שהאתר נתן לנו — אנחנו יכולים לשלוח כל בקשה שנרצה.

---

## גומרים את העבודה

עכשיו שיודעים שהטריק עובד, חוזרים עליו עם `username=administrator`:

<img width="1875" height="1004" alt="צילום מסך 2026-04-09 133115" src="https://github.com/user-attachments/assets/4a0f5179-1b63-4637-b49b-f300985b0cc8" />

מתנתקים מ-wiener ומנסים להיכנס כ-administrator עם הסיסמה החדשה שהגדרנו:

<img width="1525" height="773" alt="צילום מסך 2026-04-09 132935" src="https://github.com/user-attachments/assets/16f2a4b8-1bdb-466f-b1e5-8f07eba20ec3" />

נכנסנו. עוברים ל-Admin Panel ומוחקים את carlos:

<img width="1616" height="767" alt="צילום מסך 2026-04-09 133309" src="https://github.com/user-attachments/assets/f8e2b13b-a266-4b90-bbf9-1f54a7e44ed1" />

לאב נפתר.

---

## למה זה רלוונטי בעולם האמיתי

הפגיעות הזו היא לא תיאורטית. ב-Bug Bounty ובפנטסטים רגילים פוגשים את זה כל הזמן בצורות שונות:

- **API endpoints** שמקבלים `user_id` מהלקוח במקום לקרוא אותו מה-session — תוקף שולח `user_id` של קורבן ומשנה לו סיסמה
- **Password reset flows** שנשענים על פרמטר שהלקוח שולח לקביעת לאיזה חשבון לאפס
- **ניהול הגדרות חשבון** שלא מאמת שמי שמבצע את הפעולה הוא אותו מי שעליו מבצעים אותה

כולם נובעים מאותה בעיה יסודית: **לסמוך על קלט שבא מהלקוח לצורך קביעת זהות או הרשאה**. בצד השרת צריך לדעת מי המשתמש מה-session — לא מה שהוא טוען שהוא.

לפי OWASP זה נכנס תחת **Broken Access Control**, שנמצא במקום הראשון ב-Top 10 כבר כמה שנים. לא בכדי.

---

*PortSwigger Web Security Academy | Business Logic Vulnerabilities*
