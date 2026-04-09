<img width="1462" height="851" alt="צילום מסך 2026-04-09 151232" src="https://github.com/user-attachments/assets/2e7f22a6-8b9e-4787-9ff5-3d7869f8b507" /># פגיעות Business Logic - אימות תהליכים לא מאובטח

הסימולציה שאני עושה נמצאת באתר portswigger, שנקרא: Lab: Insufficient workflow validation

הערה מוקדמת:
```
המעבדה הזו מניחה הנחות שגויות לגבי סדר
האירועים בתהליך הרכישה. ניתן לנצל את
הפגם הזה כדי לקנות מעיל עור יקר מבלי
לשלם את המחיר המלא.

כדי לפתור את החדר, עליך לרכוש את
המוצר: "Lightweight l33t leather jacket"
```

---

## הרקע - מה זה "Insufficient Workflow Validation"?

כאשר אתר מבצע תהליך מרובה שלבים (כמו קנייה), הוא מצפה שהמשתמש יעבור דרך **כל השלבים לפי הסדר**:

```
הוסף לעגלה ➜ עבור לתשלום ➜ אמת תשלום ➜ אישור הזמנה
```

הפגיעות מתרחשת כאשר **שלב האישור הסופי אינו מאמת שאכן שלב התשלום הצליח**. אם אפשר לגשת ישירות לאישור ההזמנה — המערכת תחשוב שהתשלום עבר, גם אם לא.

> 🔴 **בחיים האמיתיים:** זו פגיעות שנמצאת באתרי מסחר אלקטרוני, מערכות הזמנות, ואפילו בסביבות banking. תוקף שמזהה תהליך כזה יכול לבצע הזמנות ללא תשלום, לקדם את עצמו לרמת מנוי שלא שילם עליה, או לעקוף בדיקות אבטחה בכל מערכת שמסתמכת על "סדר נכון" של בקשות.

---

## השלבים לפתרון

### שלב 1 - כניסה לחשבון

נכנסתי עם הפרטים שסופקו בחדר:
- **Username:** `wiener`
- **Password:** `peter`

<img width="1499" height="763" alt="צילום מסך 2026-04-09 151135" src="https://github.com/user-attachments/assets/a1a66e79-7eb3-4d28-bb2f-360135bb0aac" />

---

### שלב 2 - ניסיון קנייה ראשוני (ולמה הוא נכשל)

בדקתי את היתרה בחשבון — **$100 בלבד**.
מחיר המעיל יקר בהרבה, כך שניסיון ישיר ייכשל עם שגיאת `INSUFFICIENT_FUNDS`.

כדי **לא לפספס את מה שהמערכת עושה מאחורי הקלעים**, קניתי קודם מוצר זול שנכנס לתקציב שלי, ועקבתי אחרי בקשות ה-HTTP דרך Burp Suite:

<img width="1462" height="851" alt="צילום מסך 2026-04-09 151232" src="https://github.com/user-attachments/assets/a942484a-7e0a-42e4-a6c1-f0257f82e823" />

המוצר הזול נקנה בהצלחה:

<img width="1507" height="805" alt="צילום מסך 2026-04-09 151254" src="https://github.com/user-attachments/assets/e4bbced8-92f0-4979-85c8-cdc400a187b0" />

---

### שלב 3 - ניתוח זרימת הבקשות (כאן גילינו את הפגם)

תוך כדי המעקב הבחנתי בסדר הבקשות שהמערכת שולחת בתהליך רכישה תקין:

**א) GET /cart** — צפייה בעגלה:
```http
GET /cart HTTP/2
Host: 0a2d009403ddbe0d81f11bc000c90069.web-security-academy.net
Cookie: session=jVJMTm2Jh9xolwApLSGQ06nHMQmMA7kA
```

**ב) POST /cart/checkout** — לחיצה על "Place order":
```http
POST /cart/checkout HTTP/2
Host: 0a2d009403ddbe0d81f11bc000c90069.web-security-academy.net
Cookie: session=jVJMTm2Jh9xolwApLSGQ06nHMQmMA7kA
Content-Type: application/x-www-form-urlencoded

csrf=ry4ThJwwNOtyHrX9VvSIWZd5gvgt7kuw
```

**ג) GET /cart/order-confirmation?order-confirmed=true** — אישור הסופי:
```http
GET /cart/order-confirmation?order-confirmed=true HTTP/2
Host: 0a2d009403ddbe0d81f11bc000c90069.web-security-academy.net
Cookie: session=jVJMTm2Jh9xolwApLSGQ06nHMQmMA7kA
```

> 💡 **התובנה הקריטית:** שים לב שבקשת האישור הסופית (`order-confirmation?order-confirmed=true`) היא פשוט **GET request עם פרמטר בוליאני**. המערכת לא בודקת אם שלב ה-`/cart/checkout` אכן החזיר תשלום מוצלח — היא פשוט **סומכת על הפרמטר**. זה הפגם.

<img width="1910" height="689" alt="צילום מסך 2026-04-09 151340" src="https://github.com/user-attachments/assets/01423702-3ce8-42a4-ac4f-b3fe9ae0027d" />

---

### שלב 4 - הוספת המעיל לעגלה

שמתי את המעיל היקר (`Lightweight l33t leather jacket`) בעגלה:

<img width="1645" height="838" alt="צילום מסך 2026-04-09 151424" src="https://github.com/user-attachments/assets/9804c81b-5b06-4cf0-ab21-326bdb9e5f74" />

---

### שלב 5 - ניצול הפגם דרך Burp Repeater

הכנתי מראש (מהקנייה הקודמת) את בקשת `GET /cart/order-confirmation?order-confirmed=true` ב-Repeater.

כעת, עם המעיל בעגלה:
1. לחצתי על "Place order" — המערכת ניסתה לבדוק תשלום ונכשלה (`INSUFFICIENT_FUNDS`)
2. **שלחתי ידנית** את בקשת אישור ההזמנה ישירות מה-Repeater, **מבלי לחכות לאישור תשלום**:

<img width="1701" height="920" alt="Pasted image 20260409152313" src="https://github.com/user-attachments/assets/5674ef18-eafd-4fcf-9b73-b496c7ad1406" />

המערכת קיבלה את הבקשה, לא בדקה אם התשלום אכן עבר, ואישרה את ההזמנה:


**החדר נפתר בהצלחה. 🎉**
<img width="1510" height="769" alt="Pasted image 20260409152433" src="https://github.com/user-attachments/assets/6afb982a-bf2c-4942-8449-1af382d1c08e" />

---

## סיכום הפגם

| מה המערכת הניחה | מה שקרה בפועל |
|---|---|
| המשתמש עובר דרך כל שלבי התשלום | דילגנו ישירות לאישור |
| שלב האישור מופיע רק אחרי תשלום מוצלח | אפשר לגשת אליו ישירות |
| הפרמטר `order-confirmed=true` מגיע מהשרת | שלחנו אותו ידנית |

---

## קשר לחיים האמיתיים

פגיעות מסוג זה נמצאה בעבר בפלטפורמות מסחר אלקטרוניות אמיתיות:

- **סדר שלבים לא מאומת** — אם אתר בנה את תהליך הרכישה כ"client-side flow" וסמך על כך שהמשתמש יעבור בסדר הנכון, תוקף עם Burp/Postman יכול לשחזר בקשות ולדלג על שלבים
- **פרמטרים בוליאניים מסוכנים** — ערכים כמו `?order-confirmed=true` או `?payment-approved=1` שהשרת מקבל כ-input הם דגל אדום מיידי
- **בדיקות regression חשובות** — כל שינוי בזרימת checkout חייב לכלול בדיקה שלא ניתן לגשת לשלב N ללא עמידה בתנאי שלב N-1

> 🛡️ **כיצד מתגוננים:** השרת חייב לשמור state-side של כל עסקה — לדעת בצד השרת בלבד האם שלב התשלום הושלם, ולא לסמוך על אף פרמטר שמגיע מהלקוח.
