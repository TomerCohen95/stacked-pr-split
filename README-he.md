# סקיל לפיצול PR גדול: stacked-pr-split

לגרסה באנגלית: [English](README.md)

שינוי גדול שקשה לעשות לו review? הסקיל עוזר לסוכן הקוד לפרק אותו ל־PRים קטנים שתלויים זה בזה, עם הסבר קצר לכל שלב.

**[להורדת הסקיל כ־ZIP](https://github.com/TomerCohen95/stacked-pr-split/releases/latest/download/stacked-pr-split.zip)**

## מה הסקיל עושה

- מנתח את השינוי ומציע פיצול לשכבות: תשתית, לוגיקה, חיבור למערכת וממשק.
- מציג תוכנית ומחכה לאישור לפני ביצוע הפיצול.
- בונה סטאק בעזרת `gh stack`, כשכל PR נשען על הקודם.
- מוסיף לכל PR הסבר Before / After והכוונה ל־review.
- משתמש בסדרת commits או ב־PRים תלויים כשסטאקים אינם זמינים.

## התקנה

הורידו את קובץ ה־ZIP מהקישור למעלה וחלצו אותו. בפנים תמצאו תיקייה בשם `stacked-pr-split`.

העתיקו את התיקייה למיקום הסקילים של הסוכן:

| סוכן | מיקום התיקייה |
| --- | --- |
| סוכן Claude Code, לכל הפרויקטים | `~/.claude/skills/stacked-pr-split/` |
| סוכן Claude Code, לפרויקט אחד | `.claude/skills/stacked-pr-split/` |
| סוכן GitHub Copilot, לפרויקט אחד | `.github/skills/stacked-pr-split/` |
| סוכן אחר שתומך בסקילים | בתיקיית הסקילים שלו, בתוך תיקייה בשם `stacked-pr-split` |

ודאו שהקובץ `SKILL.md` נמצא ישירות בתוך תיקיית `stacked-pr-split`, ושהסוכן מזהה את הסקיל. לפי הסוכן, ייתכן שתצטרכו לפתוח סשן חדש או לטעון מחדש את רשימת הסקילים.

לשימוש בסטאקים של GitHub נדרשים GitHub CLI בגרסה 2.90.0 ומעלה, Git בגרסה 2.20 ומעלה, הרשאת push לריפו והרחבת `gh-stack`:

```bash
gh auth login
gh extension install github/gh-stack
```

לחלופין, למי שמעדיף Git, אפשר לשכפל ישירות למיקום הרצוי. לדוגמה, להתקנה אישית ב־Claude Code:

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/TomerCohen95/stacked-pr-split.git ~/.claude/skills/stacked-pr-split
```

## שימוש

פתחו את פרויקט הקוד בסוכן ובקשו:

> השתמש בסקיל stacked-pr-split כדי לפצל את השינוי בבראנץ' הנוכחי ל־PRים קטנים. הצג תוכנית לפני ביצוע.

הסקיל יציע סדר לפיצול, ייצור ענף גיבוי ויבקש אישור לתוכנית. ההוראות המלאות נמצאות בקובץ [SKILL.md](SKILL.md).

## קישורים

- לתיעוד GitHub: [תחילת עבודה עם stacked PRs](https://docs.github.com/en/pull-requests/get-started/stacked-prs-quickstart).
- להוראות התקנת סקילים: [Claude Code](https://code.claude.com/docs/en/skills), [GitHub Copilot](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/add-skills).
