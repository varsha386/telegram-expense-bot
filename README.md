# 💸 Telegram Expense Bot (No-Code, Make.com)

A Telegram bot that logs your daily expenses into Google Sheets. Send a short message like `200 food lunch`, and the bot saves it and replies with a confirmation. Check your monthly total with `/summary`.

Built entirely with **free, no-code tools**: Telegram Bot API, [Make.com](https://www.make.com) and Google Sheets.

---

## ✨ Features

- **Log expenses in one message**: `amount category note`
- **Monthly summary**: total spent this month
- **Undo**: delete your last entry
- **Multi-user**: each user only sees their own data (stored by `chat_id`)
- **Input validation**: helpful reply when the format is wrong
- **100% free**: runs on the Make.com Free plan

## 💬 Commands

| Send | What happens | Example reply |
|---|---|---|
| `/start` | Welcome message and command list | 👋 Hi! I'm your expense bot… |
| `200 food lunch` | Saves ₹200, category `food`, note `lunch` | ✅ Saved ₹200 · food |
| `50 travel` | Note is optional | ✅ Saved ₹50 · travel |
| `100` | Category defaults to `other` | ✅ Saved ₹100 · other |
| `/summary` | Total for the current month | 📊 November 2026 — Total spent: ₹800 |
| `/undo` | Deletes your most recent entry | ↩️ Removed ₹50 · tea |
| `/help` | Shows all commands | 📖 Commands… |
| anything else | Format hint | ❌ I didn't understand… |

**Format:** `<amount> <category> <note…>`. The amount must come first and be a number (decimals allowed).

---

## 🏗️ How It Works

```
Telegram user
     │  message
     ▼
Make.com: Telegram Bot › Watch Updates (instant)
     │
     ▼
Router
 ├─ /start   → Send welcome
 ├─ Expense  → Google Sheets › Add a Row → Send "✅ Saved"
 ├─ /help    → Send command list
 ├─ /summary → Sheets › Search Rows (this month) → Numeric Aggregator (SUM) → Send total
 ├─ /undo    → Sheets › Search Rows (latest, 1 row) → Delete a Row → Send "↩️ Removed"
 └─ Fallback → Send format hint
```

### Google Sheet structure

**Sheet `Expenses`**

| date | chat_id | amount | category | note |
|---|---|---|---|---|
| 2026-11-12 11:31:05 | 123456789 | 500 | dress | from max |

**Sheet `Users`**

| chat_id | name | budget | joined |
|---|---|---|---|

A ready-made template is in [`docs/ExpenseBot.xlsx`](docs/ExpenseBot.xlsx).

---

## 🚀 Setup Guide

### Prerequisites
- A Telegram account
- A Google account
- A free [Make.com](https://www.make.com) account

### 1. Create the Telegram bot
1. In Telegram, open **@BotFather** and send `/newbot`.
2. Choose a name and a username ending in `bot`.
3. Copy the **bot token**. ⚠️ Keep it secret and never commit it to Git.

### 2. Create the Google Sheet
1. Upload `docs/ExpenseBot.xlsx` to Google Drive.
2. Open it with Google Sheets, then **File → Save as Google Sheets**.
3. Check that the two tabs, `Expenses` and `Users`, have the headers above.

### 3a. Import the scenario (fastest)
1. In Make.com, go to **Scenarios → Create a new scenario**.
2. Click **⋯ → Import Blueprint** and choose `blueprint.json` from this repo.
3. Open each module and reconnect:
   - **Telegram Bot** modules: add a connection with your bot token.
   - **Google Sheets** modules: connect your Google account and select your `ExpenseBot` sheet.
4. Save (**Ctrl + S**).

> Connections (tokens and Google auth) are **not** included in the blueprint, so everyone must connect their own.

### 3b. Or build it manually

<details>
<summary>Click to expand module-by-module settings</summary>

> ⚠️ **Make formula rules:** don't put quotes around text inside functions. Use the `space` keyword for a space character.

**Trigger:** Telegram Bot › Watch Updates

**Router** with these routes (set each filter on the route line):

| Route | Filter |
|---|---|
| Start | `Message: Text` **Equal to** `/start` |
| Expense | `Message: Text` **Matches pattern** `^\d+(\.\d+)?(\s.*)?$` |
| Help | `Message: Text` **Equal to** `/help` |
| Summary | `Message: Text` **Equal to** `/summary` |
| Undo | `Message: Text` **Equal to** `/undo` |
| Invalid | **Fallback route = Yes** |

**Expense route: Google Sheets › Add a Row** (sheet `Expenses`)

| Column | Value |
|---|---|
| date | `{{formatDate(now; YYYY-MM-DD HH:mm:ss)}}` |
| chat_id | `{{1.message.chat.id}}` |
| amount | `{{parseNumber(get(split(trim(1.message.text); space); 1))}}` |
| category | `{{lower(ifempty(get(split(trim(1.message.text); space); 2); other))}}` |
| note | `{{join(slice(split(trim(1.message.text); space); 2); space)}}` |

Then **Telegram › Send a Text Message**:
`✅ Saved ₹<amount> · <category>`

**Summary route**
1. Google Sheets › Search Rows: `chat_id` = `{{1.message.chat.id}}` AND `date` contains `{{formatDate(now; YYYY-MM)}}`, max 500 rows
2. Tools › Numeric Aggregator: SUM of `amount`
3. Telegram › Send: `📊 {{formatDate(now; MMMM YYYY)}}` / `Total spent: ₹{{<aggregator>.result}}`

**Undo route**
1. Google Sheets › Search Rows: `chat_id` = `{{1.message.chat.id}}`, sort **descending** by `date`, max 1 row
2. Google Sheets › Delete a Row: Row number = `Row number` from step 1
3. Telegram › Send: `↩️ Removed ₹{{<search>.`2`}} · {{<search>.`3`}}`

Replace `<aggregator>` / `<search>` with the module numbers shown on your canvas.

</details>

### 4. Go live
1. At the bottom of the scenario, set scheduling to **Immediately**.
2. Turn the scenario **ON**.
3. Send `/start` to your bot. 🎉

---

## 🧪 Testing Checklist

- [ ] `/start` returns the welcome message
- [ ] `150 food dosa` returns ✅ Saved, and a row appears in the sheet
- [ ] `50 travel` works with no note
- [ ] `/summary` shows the month name and total
- [ ] `/undo` removes the latest row
- [ ] `hello` returns the format hint

## 📊 Free Plan Limits

The Make.com Free plan includes about **1,000 operations/month**. Each expense uses about 3 operations, so the bot can log roughly **300 expenses/month** at no cost. Check current limits on Make's pricing page.

## 🛣️ Roadmap

- [ ] Category-wise breakdown in `/summary`
- [ ] `/budget` command and over-budget alerts
- [ ] Automatic weekly summary every Sunday
- [ ] Spending pie chart (QuickChart.io)
- [ ] Receipt photo scanning and voice entries (Pro tier)
- [ ] Shared/group expenses

## 🔒 Security

- Never commit your **Telegram bot token** or Google credentials.
- Make.com blueprints don't include connection secrets, but review `blueprint.json` before publishing.

## 📄 License

MIT
