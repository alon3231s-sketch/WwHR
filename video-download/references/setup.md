# התקנה מודרכת של yt-dlp

המדריך להתקנה החד-פעמית. קלוד קורא את זה ומוביל את המשתמש שלב-שלב.

## עקרון העבודה מול המשתמש

פינג-פונג: משפט הסבר → פקודה אחת → מראים שהצליח → השלב הבא. דוגמה לניסוח:

> "כדי להוריד סרטונים, צריך להתקין פעם אחת כלי חינמי בשם yt-dlp - הכלי הכי נפוץ בעולם להורדת וידאו, קוד פתוח עם מיליוני משתמשים. אני אעשה הכל, זה לוקח דקה-שתיים. מתחילים?"

## שלב 1 - לבדוק אם כבר מותקן

```bash
yt-dlp --version
```

- **עובד ומחזיר גרסה** (נראית כמו תאריך, למשל `2026.07.22`) → **אין צורך בהתקנה.** לקפוץ ישר ל"שלב 3 - בדיקת ffmpeg".
- **לא מזוהה** → שלב 2 לפי מערכת ההפעלה.

לזהות את מערכת ההפעלה מהסביבה, לא לשאול את המשתמש.

## שלב 2 - התקנה

### ווינדוס (דרך winget)

```bash
winget install --id yt-dlp.yt-dlp --source winget --accept-package-agreements --accept-source-agreements --disable-interactivity
```

- לרוב מסיים תוך 1-3 דקות (הכלי קטן). אם נראה תקוע כמה דקות - לחכות, לא לבטל.
- הצלחה נראית: `Successfully installed`.
- **אימות:** `yt-dlp --version`. אם לא מזוהה - הטרמינל נפתח לפני ההתקנה:
  1. לנסות נתיב מלא: `~/AppData/Local/Microsoft/WinGet/Links/yt-dlp --version`. עובד? לשמור את הנתיב ב-`ytdlp_path` ב-state.json ולהשתמש בו תמיד.
  2. לא עובד? לעדכן state.json עם `"pending_verify": true` ולבקש מהמשתמש לפתוח מחדש את קלוד - בפעם הבאה מתחילים מהאימות.

### ווינדוס - מסלול גיבוי (אם winget לא קיים)

yt-dlp הוא קובץ אחד. מורידים ושומרים:

```bash
mkdir -p "$HOME/AppData/Local/Programs/yt-dlp"
curl -L -o "$HOME/AppData/Local/Programs/yt-dlp/yt-dlp.exe" "https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp.exe"
"$HOME/AppData/Local/Programs/yt-dlp/yt-dlp.exe" --version
```

עובד? לשמור ב-state.json: `"ytdlp_path": "C:/Users/<user>/AppData/Local/Programs/yt-dlp/yt-dlp.exe"` ומעכשיו כל פקודה משתמשת בנתיב המלא.

### מק

```bash
brew install yt-dlp
```

אם אין Homebrew - כמו במסלול הגיבוי של ווינדוס, רק עם הקובץ של מק:

```bash
mkdir -p "$HOME/.local/bin"
curl -L -o "$HOME/.local/bin/yt-dlp" "https://github.com/yt-dlp/yt-dlp/releases/latest/download/yt-dlp_macos"
chmod +x "$HOME/.local/bin/yt-dlp"
"$HOME/.local/bin/yt-dlp" --version
```

ולשמור את הנתיב ב-state.json.

## שלב 3 - בדיקת ffmpeg (חשוב לאיכות)

```bash
ffmpeg -version
```

- **מותקן** → מצוין. `"ffmpeg_available": true`. הורדות באיכות מלאה (וידאו+אודיו ממוזגים), חיתוך קטעים והמרה ל-mp3 יעבדו.
- **לא מותקן** → הסקיל עדיין עובד, אבל בלי מיזוג איכות מקסימלית, בלי mp3 ובלי חיתוך קטעים. להגיד למשתמש:

> "יש עוד כלי אחד שכדאי להתקין - ffmpeg - כדי לקבל את האיכות הכי טובה ולהמיר לאודיו. אפשר עכשיו (5-10 דקות) או אחר כך."

אם המשתמש מסכים: ווינדוס `winget install --id Gyan.FFmpeg --source winget --accept-package-agreements --accept-source-agreements --disable-interactivity` (לוקח 5-10 דקות, לפעמים נראה תקוע - זה בסדר), מק `brew install ffmpeg`. אם יש לו את הסקיל האח video-analysis - ההתקנה שם זהה ומשותפת.

לרשום את התוצאה ב-`"ffmpeg_available"` (true/false). אם false - במתכונים להשתמש בווריאציות "בלי ffmpeg".

## שלב אחרון - בדיקת אמת, ורק אז state.json

**קודם ההוכחה, אחר כך הרישום** - אם נרשום "מותקן" לפני שבדקנו והבדיקה תיכשל, הסקיל ישקר לעצמו בפעם הבאה.

1. **בדיקת אמת:** מתכון 1 מ-recipes.md (מידע בלי הורדה) על סרטון כלשהו - מוכיח שהכלי מדבר עם יוטיוב. חזר עם כותרת? עובד.
2. **רק עכשיו** לכתוב את `state.json`:

```json
{
  "setup_complete": true,
  "os": "windows",
  "ytdlp_path": "yt-dlp",
  "ytdlp_version": "2026.07.22",
  "ffmpeg_available": true,
  "install_method": "winget",
  "installed_at": "2026-07-31"
}
```

- `ytdlp_version`: הפלט של `--version` כמו שהוא (זה פורמט תאריך)
- `install_method`: `"winget"` / `"brew"` / `"direct-download"`, או `"already-installed"` אם דילגנו על ההתקנה
- `installed_at`: תאריך היום (במסלול already-installed זה תאריך הזיהוי, לא ההתקנה המקורית - וזה בסדר)
- `ytdlp_path`: `"yt-dlp"` אם מזוהה, אחרת נתיב מלא

ואז להגיד למשתמש:

> "מותקן ועובד. מעכשיו פשוט תדביק לי קישור ותגיד מה אתה רוצה - את הסרטון, רק את האודיו, או רק את הכתוביות."

ולהמשיך ישר למה שהמשתמש ביקש במקור.

## תקלות נפוצות

| תקלה | פתרון |
|---|---|
| winget מחזיר שגיאת רשת | לנסות שוב; אם נכשל - מסלול הגיבוי |
| `yt-dlp` עובד ב-cmd אבל לא בקלוד | לסגור ולפתוח את קלוד - state.json שומר את ההתקדמות |
| ההורדה איטית מאוד | נורמלי בסרטונים ארוכים; להוריד רזולוציה (מתכון 2, וריאציית 1080p) |
| "Sign in to confirm you're not a bot" / 403 | לעדכן את yt-dlp (מתכון 7) ולנסות שוב אחרי כמה דקות |
