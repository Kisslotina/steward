# Taxonomy & routing map

Single taxonomy for `Inbox.Type` and where each type goes. Keep in sync with the Notion bases
(see `conventions.md`) and `docs/architecture.md` (§6).

## Types (Inbox.Type)

`task · reminder · reference · goal · thought · event · expense · purchase · unsure`

## Routing map

| Type | Destination | Notes |
|---|---|---|
| task | **Tasks** | Type=Work/Personal, Priority, Do date, Assignee; Tag=From Daily if from Today; Executor=Me/Auto |
| reminder | **Tasks** | Tag=Reminder + date |
| reference | **Knowledge / Tech notes** | a fact/reference, NOT a task |
| goal | **Goals** | Horizon, Area, Target date |
| thought | **Content Ideas** | Platform/Status optional |
| event | **Outbox** (Type=calendar, Handler=Steward (MCP)) | enqueue; the executor routine creates the calendar event via MCP |
| expense | (no destination yet) | set Type, leave New, note it in the report — Finance not connected |
| purchase | (no destination yet) | set Type, leave New, note it in the report — Wish list not connected |
| unsure | **Not Recognized** | Note + Reason + Source; do not guess — leave for the user |

## Rules

- Do not split "partially recognized": either file it confidently or send the whole thing to Not Recognized with a reason.
- `reference` ≠ action: knowledge is filed separately from tasks.
- Anything doubtful → `unsure` / Not Recognized; never guess a category.
- The sort routine never executes outbound work — it enqueues an **Outbox** row with a `Handler`: `notify` → Concierge Gateway; `calendar/doc/sheet/other` → Steward (MCP, executor routine).

## Completion / update notes (not a new record)

Some notes mark an EXISTING item done (e.g. "closed task X", "finished goal Y"). These do NOT create
a new record — `sort-inbox` matches the existing item and updates its status:
- one confident match → set Done / Status=Done; write to the audit log; enqueue an Outbox `notify`
  row (Handler=Gateway, so it logs to Telegram); Inbox.Target = "closed: <base>/<title>".
- no/ambiguous match → Not Recognized ("completion note, no confident match"). Never guess.
