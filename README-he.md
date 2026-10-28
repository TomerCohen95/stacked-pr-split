# stacked-pr-split

סקיל לסוכני קוד (Claude Code, Codex, Copilot ודומים) שלוקח שינוי גדול ומפרק אותו לסטאק של PRים קטנים שנבנים אחד על השני, עם `gh stack` של GitHub.

מה הוא עושה בפועל:
1. מנתח את השינוי הגדול
2. מתכנן פיצול לשכבות (יסודות → לוגיקה → חיבור → UI)
3. מחכה לאישור לפני שדוחף
4. בונה את הסטאק — כל PR יושב על הקודם
5. מוסיף לכל PR הסבר קצר (Before / After) כדי שה־review יהיה קל

אם אין stacked PRs (לא GitHub, fork, GHES), בונה במקום זה סדרת commits מסודרת או PRים תלויים.

התקנה: `gh extension install github/gh-stack`, ואז להעתיק את התיקייה `stacked-pr-split` לתיקיית הסקילים של הסוכן (למשל `~/.claude/skills/` או `.github/skills/` בריפו).
