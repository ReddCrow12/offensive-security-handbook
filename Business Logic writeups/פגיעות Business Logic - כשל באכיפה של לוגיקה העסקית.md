
הסימולציה שאני עושה נמצאת באתר portswigger, שנקרא: Lab: Flawed enforcement of business rules


הערה ורקע:
```
לחדר הזה יש פגם לוגי בתהליך הרכישה שלה
כדי לפתחור את המעבדה, עליכם לנצל את הפגם הזה כדי לקנות
"Lightweight l33t leather jacket"

תתחבר בעזרת החשבון טסט הזה:
wiener:peter
```

### הפתרון המלא:

<img width="1738" height="797" alt="Pasted image 20260331163937" src="https://github.com/user-attachments/assets/c280a7ef-b622-43a8-9c68-9e87a49a74e6" />

לפני שאנחנו נרשמים אנחנו רואים משהו מאוד מעניין, באתר לפני ההרשמה כתוב `New customers use code at checkout: NEWCUST5`
באתר למטה אפשר לראות שדה מייל להרשמה לעלון הדיגיטלי שלהם, לאחר ההרשמה אנחנו מקבלים עוד שובר קניה: `Use coupon SIGNUP30 at checkout!`
אולי נוכל לנצל את זה בהמשך, צריך לזכור את זה לאחר כך.

לאחר שנכנסנו עם שם משתמש והסיסמה של חשבון הטסט, אנחנו רואים שיש לנו 100$ כמו פעם שעברה, אנחנו צריכים לקנות את המעיל עור שוב,
מן הסתם שהפעם אותה פגיעות לא תעבוד אז נצטרך לחשוב על משהו..

<img width="1537" height="751" alt="Pasted image 20260331165242" src="https://github.com/user-attachments/assets/5d76dd2f-920e-4bf4-902a-84ebbc2b5970" />

אחרי ששמתי את 2 הקופונים עלה לי רעיון אולי ננסה להשים את אותו קופון פעמיים, ואחרי שבדקתי את `SIGNUP30` ראיתי את הכיתוב `Coupon already applied` אחרי זה ניסיתי את הקופון `NEWCUST5` וזה הצליח ובפעם השניה, ניסיתי שוב וזה לא הצליח,

כנראה המערכת מזהה כמה פעמים שמת את אותו קופון ברצף, נשים את הקופונים כל פעם בסדר אחד אחרי השני

<img width="1651" height="895" alt="Pasted image 20260331170141" src="https://github.com/user-attachments/assets/ab585c24-bb74-4f55-aa27-1643891ae7b3" />

לאחר שניצלנו את הקופונים אנחנו רואים שבאתר שהמחיר הכולל בעגלה הוא 0$, לאחר שלוחצים `place order` אנחנו רואים שפתרנו את החדר הזה.

<img width="1587" height="787" alt="Pasted image 20260331171132" src="https://github.com/user-attachments/assets/a3b16fc2-4ec9-4ba0-861b-520afcfa49ca" />
