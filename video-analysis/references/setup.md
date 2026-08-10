# התקנה מודרכת של ffmpeg

המדריך המלא להתקנה החד-פעמית. קלוד קורא את זה ומוביל את המשתמש שלב-שלב.

## עקרון העבודה מול המשתמש

פינג-פונג: משפט הסבר → פקודה אחת → מראים שהצליח → השלב הבא.
בעברית פשוטה. דוגמה לניסוח טוב:

> "כדי שאוכל לנתח סרטונים, צריך להתקין פעם אחת כלי חינמי בשם ffmpeg - זה הכלי הסטנדרטי בעולם לעבודה עם וידאו. אני אעשה הכל, אתה רק תראה אותי עובד. זה לוקח 5-10 דקות. מתחילים?"

## שלב 1 - לבדוק אם כבר מותקן

```bash
ffprobe -version && ffmpeg -version
```

- **שתי הפקודות עובדות ומחזירות גרסה** → לדלג על כל ההתקנה. לקפוץ ישר ל"שלב אחרון".
- **אחת מהן לא מזוהה** → להמשיך לשלב 2 לפי מערכת ההפעלה. (בודקים את שתיהן - הסקיל משתמש גם ב-ffprobe לניתוח וגם ב-ffmpeg לחילוץ, ומחשב עם אחת בלי השנייה יעבור התקנה מלאה.)

לזהות את מערכת ההפעלה מהסביבה (Platform בסביבת העבודה, או `uname` מול `$env:OS`). לא לשאול את המשתמש אם אפשר לזהות לבד.

## שלב 2 - ווינדוס: התקנה דרך winget

winget הוא מנהל ההתקנות הרשמי של מיקרוסופט, מגיע מובנה בווינדוס 10 ו-11. הוא מוריד את הבנייה הרשמית של ffmpeg (Gyan.FFmpeg - הבנייה שאתר ffmpeg.org מפנה אליה).

להגיד למשתמש לפני ההרצה: **"ההתקנה לוקחת 5-10 דקות ולפעמים נראית תקועה - זה בסדר, היא עובדת."**

```bash
winget install --id Gyan.FFmpeg --source winget --accept-package-agreements --accept-source-agreements --disable-interactivity
```

הערות מניסיון אמיתי:

- הפקודה יכולה לרוץ **מעבר ל-10 דקות**. להריץ עם timeout ארוך או ברקע. אם עברו 10 דקות בלי פלט - לא לבטל, לחכות. בהרצה אמיתית היא סיימה בהצלחה אחרי שנראתה תקועה.
- ההצלחה נראית ככה בסוף הפלט: `Successfully installed` + `Command line alias added: "ffmpeg"`.

### אחרי ההתקנה - אימות

```bash
ffprobe -version
```

- **עובד** → לשלב האחרון.
- **לא מזוהה** → זה מצב צפוי ומוכר: הטרמינל הנוכחי נפתח לפני ההתקנה ולא מכיר עדיין את הפקודה החדשה. הפתרון בשלושה מדרגות:
  1. לנסות נתיב מלא: `"$env:LOCALAPPDATA\Microsoft\WinGet\Links\ffprobe.exe" -version` (או ב-bash: `~/AppData/Local/Microsoft/WinGet/Links/ffprobe`). אם עובד - לשמור את הנתיב המלא ב-state.json (שדות `ffmpeg_path`/`ffprobe_path`) ולהשתמש בו תמיד.
  2. אם גם זה לא - לבקש מהמשתמש לסגור ולפתוח מחדש את קלוד/הטרמינל. **לפני כן לעדכן את state.json** עם `"install_method": "winget"` ו-`"setup_complete": false` + להוסיף שדה `"pending_verify": true` - ככה בפעם הבאה קלוד ידע להתחיל מהאימות ולא מההתחלה.
  3. אם winget עצמו לא קיים במחשב (נדיר) → מסלול הגיבוי למטה.

### מסלול גיבוי לווינדוס (רק אם winget לא קיים)

```bash
mkdir -p "$HOME/AppData/Local/Programs/ffmpeg"
curl -L -o "$HOME/AppData/Local/Programs/ffmpeg/ff.zip" "https://www.gyan.dev/ffmpeg/builds/ffmpeg-release-essentials.zip"
cd "$HOME/AppData/Local/Programs/ffmpeg" && unzip -q ff.zip && rm ff.zip
```

אחרי החילוץ נוצרת תיקייה כמו `ffmpeg-7.1-essentials_build`. הבינארים ב-`<תיקייה>/bin/`. לא לשחק עם PATH של המערכת - פשוט לשמור את הנתיבים המלאים ב-state.json:

```json
"ffmpeg_path": "C:/Users/<user>/AppData/Local/Programs/ffmpeg/ffmpeg-7.1-essentials_build/bin/ffmpeg.exe",
"ffprobe_path": "C:/Users/<user>/AppData/Local/Programs/ffmpeg/ffmpeg-7.1-essentials_build/bin/ffprobe.exe"
```

ומעכשיו כל פקודה במתכונים משתמשת בנתיב המלא במקום `ffmpeg`.

## שלב 2 - מק: התקנה דרך Homebrew

```bash
brew install ffmpeg
```

- אם `brew` לא מזוהה - למשתמש אין Homebrew. שתי אפשרויות, לשאול את המשתמש מה מעדיף:
  1. להתקין Homebrew (הסטנדרט במק, שימושי להמון דברים): `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"` - ידרוש סיסמת מחשב מהמשתמש.
  2. בלי Homebrew: להוריד בינארי סטטי מ-evermeet.cx (ffmpeg + ffprobe בנפרד), לחלץ, `chmod +x`, ולשמור נתיבים מלאים ב-state.json כמו במסלול הגיבוי של ווינדוס.

אחרי ההתקנה: `ffprobe -version` לאימות.

## שלב אחרון - לעדכן את state.json (חובה, לא לדלג)

זה הצעד שהופך את ההתקנה לחד-פעמית. לכתוב את `state.json` בתיקיית הסקיל:

```json
{
  "setup_complete": true,
  "os": "windows",
  "ffmpeg_path": "ffmpeg",
  "ffprobe_path": "ffprobe",
  "ffmpeg_version": "8.1.2",
  "install_method": "winget",
  "installed_at": "2026-07-31"
}
```

- `os`: `"windows"` או `"mac"`
- `ffmpeg_version`: **המספר בלבד**, בלי סיומות. הפלט `ffprobe version 8.1.2-full_build-www.gyan.dev` נרשם כ-`"8.1.2"`
- `install_method`: `"winget"` / `"brew"` / `"direct-download"` / `"already-installed"`
- `ffmpeg_path`/`ffprobe_path`: `"ffmpeg"`/`"ffprobe"` אם הפקודות מזוהות, אחרת נתיב מלא

ואז להגיד למשתמש משהו כמו:

> "מותקן ועובד. מעכשיו, בכל פעם שתבקש ממני לנתח סרטון - פשוט תגיד 'מה יש בסרטון X' ואני אגש ישר לעבודה, בלי ההתקנה הזאת."

ולהמשיך ישר למה שהמשתמש ביקש במקור (אם ביקש ניתוח - לנתח עכשיו).

## תקלות נפוצות

| תקלה | פתרון |
|---|---|
| winget מחזיר שגיאת רשת | לנסות שוב פעם אחת; אם נכשל - מסלול הגיבוי |
| "Access denied" בהתקנה | להריץ בלי `--scope machine` (ברירת המחדל היא user, לא צריך הרשאות מנהל) |
| הפקודה עובדת ב-cmd אבל לא בקלוד | לסגור ולפתוח את קלוד מחדש - state.json שומר את ההתקדמות |
| מק: brew תקוע על "Updating Homebrew" | לחכות. brew מעדכן את עצמו לפני ההתקנה, זה נורמלי |
