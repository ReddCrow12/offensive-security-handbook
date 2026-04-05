הסימולציה שאני עושה נמצאת באתר portswigger, שנקרא: Lab: Excessive trust in client-side controls

הערות ורקע על המטרה:
```
האתר הזה לא מאמת כמו שצריך את הקלט של המשתמש.
ניתן לנצל פגיעות לוגית בתהליך הרכישה שלה כדי לקנות
דברים במחיר לא הלא נכון,
בשביל לפתור את החדר הזה עלייך לקנות 'Lightweight l33t leather jacket' עם הניצול הנ''ל
:אתה יכול להתחבר לחשבון הטסט שלך

wiener:peter
```

### הפתרון המלא:

![[Pasted image 20260331151015.png]]

דבר ראשון אנחנו צריכים להתחבר עם המשתמש שנתנו לנו לבדיקה.
ולאחר ההתחברות אתם יכולים לראות שיש לי קרדיט לבדיקה של 100$

![[Pasted image 20260331151553.png]]

עכשיו אני ינסה לקנות את המעיל עור שדובר עליו בהקדמה של החדר
תזכרו שיש לי 100$, אבל אני בכל זאת ינסה לעשות מניפולציית מחיר
על המערכת. אז נכנסתי לדף של המעיל עור והוספתי אותו לעגלה
וכשאני לוחץ על הכפתור `place order` למרות שאני רואה ש `Total = 1335.00$` ויש לי בחשבון רק 100$ וזה מראה את ההודעה הבאה:
ו- `Not enough store credit for this purchase`

אבל לאחר שימוש בכלי burpsuite אני רואה שמתי שאני נכנס לעגלה
מתקבלת הודעת POST שבתוכה נמצא פרמטר שנקרא `price`:

``` http
POST /cart HTTP/2
Host: 0ac2003f032ae3f6803a580900560049.web-security-academy.net
Cookie: session=ijh8EixYn0h4l7HBBQnSg1xhYaZw2IP2
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 49
Origin: https://0ac2003f032ae3f6803a580900560049.web-security-academy.net
Referer: https://0ac2003f032ae3f6803a580900560049.web-security-academy.net/product?productId=1
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers

productId=1&redir=PRODUCT&quantity=1&price=133700
```

אם אני ינסה להיכנס מחדש לדף העגלה ויעשה intercept אני אולי הוכל לשנות את המחיר לכמה שאני רוצה על ידי ניצול של המערכת שסומכת יותר מידי על הלקוחות שלה.
בדוגמא הזאת אני ישנה את המחיר לסנט אחד בלבד, ואנסה לקנות את המוצר.

![[2026-03-31 15-39-00.mp4]]

כמו שראיתם בסרטון הניצול צלח, ולאחר אישור התשלום פתרתי את החדר
מה שראינו פה היה טעות אנוש קטנה שיכולה להוביל עסק לאסון כלכלי
רק כי המפתחים היו "עצלנים" לחשוב מה היה קורה.

אומנם כיום הפגיעות היא יותר קשה למצוא אבל היא על אותו עיקרון ובסיס
וצריך תמיד לוודא ולאבטח את הדברים שחשובים לנו.

![[Pasted image 20260331154650.png]]

