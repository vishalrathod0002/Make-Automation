# 🎉 HR Automation — Employee Birthday Wish Workflow

A Make.com automation that sends personalized, beautifully designed birthday emails to employees — pulling data from Google Sheets, fetching photos from Google Drive, generating a warm birthday message with Groq's LLaMA 3.3, and delivering it in a styled dark/gold HTML email card.

---

## ✨ Features

- **Fully automated** — runs on a daily schedule, no manual triggering
- **Smart filtering** — only processes `Active` employees whose `DOB` matches today's date (month + day)
- **AI-generated messages** — unique birthday text per employee via Groq's free-tier LLaMA 3.3 70B model
- **Styled HTML email** — dark, gold-accented birthday card with a glowing photo frame, name, department badge, and message card, plus a graceful fallback message if the AI call fails
- **Zero ongoing AI cost** — Groq's free tier covers text generation; no paid image generation required
- **Row tracking** — updates the sheet after each send

---

## 🏗️ Architecture

```
Google Sheets — Filter Rows (Status = Active)              [module 14]
        │
        ▼
Filter — DOB matches today's date (MM-DD)
        │
        ▼
Google Drive — Download employee photo                     [module 5]
        │
        ▼
HTTP → Groq (llama-3.3-70b-versatile) — Generate message    [module 41]
        │
        ▼
Send Email — styled HTML card, photo inline via CID         [module 10]
        │
        ▼
Google Sheets — Update Row                                  [module 21]
```

Trigger this scenario on a **daily schedule** (set via the clock icon in Make's scenario editor — "At regular intervals" → Days).

---

## 📊 Google Sheet Structure

| Column | Field | Notes |
|---|---|---|
| A | `EmployeeName` | Full name |
| B | `Email` | Recipient address |
| C | `DOB` | Full date, must be a real **Date** type (not text) — the MM-DD filter depends on this |
| D | `Department` | Used in the AI prompt and the email badge |
| E | `PhotoDriveFileId` | Google Drive **file ID only** — not the full share URL, not a folder link |
| F | `LastWishedYear` | Updated automatically after each send |
| G | `Status` | Must be exactly `Active` for a row to be processed |

---

## ⚙️ Setup

### 1. Google Sheet
Create the sheet matching the structure above. Format column C as a real Date type (Format → Number → Date) — text-formatted dates will silently break the birthday-matching filter.

### 2. Groq API key (free, no card required)
1. Sign up at [console.groq.com](https://console.groq.com)
2. **API Keys** → create one
3. Paste it into module 41's `Authorization: Bearer` header

### 3. Import into Make.com
1. Make.com → new scenario → **Import Blueprint** → upload the provided `.json`
2. Reconnect Google Sheets, Google Drive, and Gmail (Make will prompt for each on import)
3. Set the daily schedule via the clock icon

### 4. (Optional) Decorative frame
The current email uses CSS-only decoration (gradients, glow effects, emoji) — no external image generation required, so no additional setup or ongoing cost.

---

## 📁 Files in this repo

| File | Purpose |
|---|---|
| `hr_automation_blueprint.json` | Make.com scenario — import this directly |
| `employee_birthday_sheet.xlsx` | Starter spreadsheet template matching the required columns |

---

## ⚠️ Known Issues / Things to Verify

- **Status gets overwritten on send.** Module 21 currently writes `"Wished"` into the `Status` column (G) after sending. Since module 14's filter requires `Status = Active`, this means an employee who's been wished **will not be included in future runs** — including next year's birthday — unless `Status` is manually reset back to `Active`. Decide whether this is the intended behavior (e.g. manual yearly re-activation) or whether module 21 should only update `LastWishedYear` (column F) and leave `Status` untouched.
- **Response array indices** (`choices[1]` in the Groq call) should be verified against a live test run — Make's array indexing can behave differently between native modules and raw HTTP JSON responses.
- **DOB must be a genuine Date-typed cell.** A visually correct but text-formatted date will cause the birthday filter to silently never match.

---

## 🔧 Troubleshooting

| Symptom | Likely Cause |
|---|---|
| `[404] File not found` on photo | `PhotoDriveFileId` contains a full URL instead of just the file ID |
| `[403] Export only supports Docs Editors files` | `PhotoDriveFileId` points to a folder, not a file |
| Filter never matches | DOB column stored as text, not a real Date type |
| Employee never gets wished again next year | See "Status gets overwritten" above |
| Empty message in email | Groq call failed or returned unexpected shape — the `ifempty()` fallback in the HTML should catch this; verify `choices[1]` index against a real test run |

---

## 📝 License

Internal / personal use — adapt as needed for your organization.
