הסימולציה שאני עושה נמצאת באתר portswigger, שנקרא: Lab: Low-level logic flaw

הערה מקדימה:
```
החדר הזה אינו מאמת קלט משתמש, וניתן לנצל
פגם לוגי בתהליך הרכישה שלה כדי לקנות
דברים במחיר אחר. כדי לפתור את
החדר עליכם לקנות "Lightweight l33t leather jacket"

ניתן להתחבר לחשבון הטסט באמצעות פרטי הכניסה:
wiener:peter
```

### הפתרון המלא:

<img width="1625" height="659" alt="Pasted image 20260401135919" src="https://github.com/user-attachments/assets/9804e0ef-39ca-49ba-aa50-e861166dd811" />

אחרי שהתחברתי לאתר אני רואה שיש לי 100$ והמעיל עולה יותר מהתקציב של המשתמש טסט. 
אני ישחק טיפה עם ה- burpsuite במטרה לתפוס איזו בקשה שנראה שאפשר לנצל אותה.

ואז אני רואה את הבקשה הזאת..
<img width="1636" height="856" alt="Pasted image 20260401140546" src="https://github.com/user-attachments/assets/4947bfae-a0f6-4dbd-aada-af2b0d7b4f8c" />


זאת הבקשה שמעבירה את המעיל עור לעגלת קניות לסיום הרכישה
אנחנו רואים שיש פרמטר בשם `quantity=1` שאומר כמה יחידות של המוצר להעביר לעגלת קניות

``` http
POST /cart HTTP/2
Host: 0a0000060424a800807dbc3f00310099.web-security-academy.net
Cookie: session=g6LT84cFbUATy7GrkbvwFBtLuzqx2ZXh
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 37
Origin: https://0a0000060424a800807dbc3f00310099.web-security-academy.net
Referer: https://0a0000060424a800807dbc3f00310099.web-security-academy.net/product?productId=1
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers

productId=1&redir=PRODUCT&quantity=99
```

אחרי שניסיתי לשנות את המספר היחידות להזמין ל- 100 יחידות ראיתי שהמגבלה היא 99 וזה עבד, ישר אחרי שלחתי את הבקשה ל `repeater` ואחרי ששלחתי עוד כמה פעמים אני רואה שהצלחתי לעקוף את כמות היחידות שאפשר להזמין:

<img width="1237" height="678" alt="Pasted image 20260401141428" src="https://github.com/user-attachments/assets/9531014e-a777-488b-83cc-5df976489f47" />


ואז חשבתי לעצמי..
מה יקרה אם אני ישלח מספר בקשות כל כך גבוהה עד שמספר היחידות יוביל ל ERROR באפליקציה והיא איך היא תתנהג אחרי ההצפה הזאת?
אז שלחתי את הבקשה ל `Intruder` הגדרתי `Null payload` וכמות פעמים לשליחה לשרת = 300

<img width="1494" height="679" alt="Pasted image 20260401142152" src="https://github.com/user-attachments/assets/54bd945e-672e-4229-80df-bbb39487ab47" />

בום! אחרי כמה בקשות המערכת הגיעה לקצה בקשות ההזמנה
ומרוב שהצפנו את השרת, המחיר הגיעה גם לקצה ומשם נהפך לשלילי:

<img width="1477" height="645" alt="Pasted image 20260401143423" src="https://github.com/user-attachments/assets/e95b992c-6bc8-4932-ac00-a6ad768e0596" />

אחרי שעצרתי את ה `intruder` גיליתי שאם אני מעביר ידנית עוד בקשות 
הוא מוסיף למספר הכולל של עלות ההזמנה ולא מוריד אותו אלא ממשיך להתקרב ל-0.
אני ישלח כמה בקשות בצורה ידנית ואני יוסיף מוצרים אחרים עד שנגיע ל-0 ונוכל לנצל את הלוגיקה של הקוד על מנת שנצליח לפתור את החדר ולקנות את המעיל עור.

<img width="1500" height="739" alt="Pasted image 20260401145741" src="https://github.com/user-attachments/assets/b5fed854-1dbf-41aa-b669-d8519edb9f48" />

הצלחנו לקנות מספר מופרז של פריטים מהחנות רק ב8.64$, כל מה שנשאר לעשות זה ללחוץ `place order` ופתרנו את החדר. 

<img width="1423" height="591" alt="Pasted image 20260401145946" src="https://github.com/user-attachments/assets/208d7cf5-ce69-4e4b-ae14-3ed43e9d43f2" />
