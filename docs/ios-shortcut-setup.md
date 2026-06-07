# iOS Shortcut → Notion Inbox — setup

> Goal: trigger on iPhone → dictate → text is written into the Notion **Inbox** database.
> Inbox fields: Note (Title) / Status / Type / Target / Date.

Three capture triggers point to the **same** shortcut: **Hey Siri "Inbox"**, the **Action Button**,
and **Back Tap** (double tap on the back). On-device Apple speech-to-text turns voice into text, so
Notion only ever stores text (no audio, no token cost for transcription).

> Language note: when launched via Siri, the dictation language follows the **Siri language**, not
> the shortcut's setting. To dictate non-English notes by voice, set Siri to that language. Via
> Action Button / Back Tap, the language comes from the Dictate Text action.

---

## Part 1. Notion — integration & access (once, from a computer)

### 1.1 Create an Internal integration
1. Open https://www.notion.so/my-integrations
2. **New integration** → Type: **Internal**.
3. Name: `Steward Capture`. Associated workspace: your workspace.
4. Capabilities: **Read content** + **Insert content** + **Update content** are enough.
5. **Save** → copy the **Internal Integration Secret** (starts with `ntn_...`). Keep it private.

### 1.2 Connect the integration to the Inbox database
> Without this step the Shortcut returns 404.
1. Open the **Inbox** page in Notion.
2. Top-right of the database → **•••** → **Connections** → add **Steward Capture**.

### 1.3 Database ID
Open the Inbox database as a full page, copy the URL: the 32-char string between the last `/` and
`?` is the Database ID. Put it in your `.env` as `NOTION_INBOX_DB_ID`.

---

## Part 2. iPhone Shortcut (Shortcuts app)

### 2.1 Create the shortcut
1. Shortcuts → **+** → name it `Inbox` (short, single word — Siri recognizes it reliably).

### 2.2 Actions (add in order)

**Action 1 — Dictate Text**
- Language: your note language (e.g. English).
- Stop Listening: **After Pause**.

**Action 2 — Text** (the JSON body)
```json
{
  "parent": { "database_id": "YOUR_DATABASE_ID" },
  "properties": {
    "Note":   { "title": [ { "text": { "content": "DICTATION" } } ] },
    "Status": { "select": { "name": "New" } }
  }
}
```
Delete the word `DICTATION` (keep the empty `""`) and insert a **Magic Variable** → the
**Dictated Text** result. Must be the blue variable pill, not typed text.

**Action 3 — Get Contents of URL**
- URL: `https://api.notion.com/v1/pages` (plain text, no variables in this field)
- Method: **POST**
- Headers:
  - `Authorization` = `Bearer ntn_YOUR_TOKEN` (note the space after `Bearer`)
  - `Notion-Version` = `2022-06-28`
  - `Content-Type` = `application/json`
- Request Body: **File** → select the **Text** variable from Action 2.

**Action 4 (optional) — Show Notification**: `Saved to Inbox ✅`

### 2.3 Wire up the triggers
- **Action Button:** Settings → Action Button → swipe the carousel to **Shortcut** → pick `Inbox`.
- **Back Tap:** Settings → Accessibility → Touch → Back Tap → **Double Tap** → `Inbox`.
- **Siri:** "Hey Siri, Inbox" works automatically once the shortcut is named `Inbox`.

---

## Verify
Run the shortcut, dictate "test note" → a row should appear in Inbox with Note = "test note",
Status = New.

## Common errors
- **401** — bad/missing token, or missing `Bearer ` prefix.
- **404** — integration not connected to the database (step 1.2) or wrong Database ID.
- **400** — property name mismatch (case: `Note`, `Status`), missing `New` option, or the variable
  was typed as text instead of inserted as a Magic Variable.

## Security
The token sits in the Shortcut in plain text. Give the integration access to the **Inbox database
only**, and don't share the Shortcut.
