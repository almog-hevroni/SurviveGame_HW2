<div dir="rtl">

# 🧠 Survive Game – הנדסה הפוכה 

פרויקט זה עוסק בהנדסה לאחור של אפליקציית משחק שפורקה מקובץ APK.  
המטרה הייתה להבין את אופן הפעולה של האפליקציה, לתקן שגיאות, ולהריץ את המשחק בהצלחה לפי הלוגיקה שבקוד.

---

## 🔧 תהליך העבודה

### 1. פירוק הקוד (Decompiling)
- קיבלנו קובץ APK.
- ביצענו לו decompile
- חיפוש אחר קבצים: קבצי Java, XML ו־AndroidManifest.

### 2. ייבוא לקוד מקור בפרויקט Android Studio
- יבאתי את הקבצים הבאים:
    - `AndroidManifest.xml`
    - `Activity_Menu.java`
    - `Activity_Game.java`
    - `activity_menu.xml`
    - `activity_game.xml`

---

## 🛠️ תיקונים שבוצעו בקובץ AndroidManifest.xml

| בעיה | פתרון |
|------|--------|
| אי התאמה בשם החבילה | תואם לשם המחלקות בקוד (`com.example.survivegame_hw2`) |
| חסר `android:exported="true"` | נוסף ל־Activity עם `intent-filter` |
| שימוש ב־platformBuildVersionCode / Name | הוסרו — אינם חוקיים |
| `appComponentFactory` שדורש API 28+ | הוסר כדי לא לדרוש minSdk גבוה |
| `targetSdkVersion` נמוך מדי | עודכן ל־34 בהתאם לדרישות Google Play |

---
## 🐞 פתרון שגיאות בקוד

### 📄 Activity_Game.java

#### ❌ שגיאה: שימוש לא תקין ב-Toast

- **הסבר**: הקוד כלל קריאה ל־`Toast.makeText()` עם ערך מספרי קשיח (`1`) כמשך ההצגה של ההודעה.  
  אמנם `1` שווה ל־`Toast.LENGTH_LONG`, אך לפי ה־API של אנדרואיד נדרש שימוש בקבועים בלבד (`Toast.LENGTH_SHORT` או `Toast.LENGTH_LONG`).  
  שימוש בערכים מספריים קשיחים גורם לשגיאות Lint ואינו תקני.

- **קוד לפני התיקון**:
  ```java
  Toast.makeText(this, "Survived in " + state, 1).show();
  Toast.makeText(this, "You Failed ", 1).show();
  ```

- **קוד אחרי התיקון**:

```java
Toast.makeText(this, "Survived in " + state, Toast.LENGTH_SHORT).show();
Toast.makeText(this, "You Failed ", Toast.LENGTH_SHORT).show();
```

### 📄 Activity_Menu.java

#### ❌ שגיאה: `Cannot resolve symbol 'url'`
- **הסבר**: לא הוגדר ערך בשם `url` בקובץ `strings.xml`.
- **פתרון**:
  הוספתי את השורה הבאה ל־`res/values/strings.xml`:
  ```xml
  <string name="url">https://pastebin.com/raw/T67TVJG9</string>

#### ❌ תווים בלתי נראים (Zero-width characters) בכתובת ה־URL

- **הסבר**: תווים מוסתרים חדרו לכתובת ה־URL במהלך ההעתקה. תווים אלו אינם נראים לעין אך גורמים לקריסת הקריאה מהשרת ולשגיאת חיבור

  לדוגמה, בתמונה הבאה מתוך Android Studio ניתן לראות את התווים הבלתי נראים שהופיעו לאחר הדבקת הכתובת :

![Zero Width Characters in URL](./wrong_url.png)

- **פתרון**:
  הקלדתי ידנית את הכתובת שמצאתי בתקייה

## 🕹️ איך עובד המשחק?

לאחר מעבר על הקוד וניתוח שלו הבנתי שהאפליקציה כוללת שני מסכים עיקריים:

---

### 1. `Activity_Menu` – מסך פתיחה

1. המשתמש מזין תעודת זהות באורך 9 ספרות בשדה טקסט.
2. מתבצעת קריאה לשרת בעזרת הפונקציה:
   ```java
   getJSON()
    ```
3. התגובה מהשרת היא מחרוזת של מצבים (states) שמכילה שמות של מדינות, מופרדים בפסיק:
   California,Texas,Florida,New York,Illinois,Pennsylvania,Ohio,Washington,Michigan,Arizona
   המחרוזת מפוצלת למערך מחרוזות באמצעות הפונקציה:
    ```java
    String[] allStates = data.split(",");
    ```
   כתוצאה מכך נוצר מערך בגודל 10, כאשר כל אינדקס מייצג מדינה לפי הסדר:
0. California
1. Texas
2. Florida
3. New York
4. Illinois
5. Pennsylvania
6. Ohio
7. Washington
8. Michigan
9. Arizona

4. הספרה השביעית בת"ז (במיקום id.charAt(7)) משמשת כאינדקס לבחירת מצב (state) מתוך המחרוזת

### 2. `Activity_Game` – מסך המשחק

1. ישנם 4 כפתורים: ⬅️ ➡️ ⬆️ ⬇️ (left, right, up, down)
2. הקוד מבצע חישוב:
    ```java
   steps[i] = Character.getNumericValue(id.charAt(i)) % 4;
    ```
3. חץ מתאים → תרגום לערך בין 0–3 → כל ספרה בת"ז
4. מופיעה הודעה "Survived in {state}" → אם הכל נכון

#### 🔎 דוגמה: תעודת זהות `206843369`

חישוב הצעדים מתבצע לפי הפעולה `mod 4` על כל ספרה בתעודת הזהות:

| אינדקס בת"ז | ספרה | ספרה % 4 | חץ תואם    |
|--------------|--------|------------|--------------|
| 0            | 2      | 2          | ⬆️ Up        |
| 1            | 0      | 0          | ⬅️ Left      |
| 2            | 6      | 2          | ⬆️ Up        |
| 3            | 8      | 0          | ⬅️ Left      |
| 4            | 4      | 0          | ⬅️ Left      |
| 5            | 3      | 3          | ⬇️ Down      |
| 6            | 3      | 3          | ⬇️ Down      |
| 7            | 6      | 2          | ⬆️ Up        |
| 8            | 9      | 1          | ➡️ Right     |

---

**סדר הלחיצות הנדרש:**
⬆️ ⬅️ ⬆️ ⬅️ ⬅️ ⬇️ ⬇️ ⬆️ ➡️

