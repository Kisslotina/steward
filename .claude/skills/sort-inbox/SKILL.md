---
name: sort-inbox
description: Classifies and files raw Inbox records into the typed Notion structure, applies completion notes that close existing items, and enqueues outbound work (notifications, external actions) to the Outbox. Use after capturing notes, or whenever the user asks to sort/process the Inbox or clear new captures. Runs only after sweep-daily-notes has finished. Reads Inbox rows with Status=New, decides new-vs-completion, routes or updates accordingly, sets Status=Sorted and Target.
---

# Sort Inbox

Processes every Inbox record with `Status=New` by the single taxonomy. Source-independent: by now
everything is just Inbox rows (voice or Daily notes). Runs ONLY after `sweep-daily-notes` has fully
completed.

## When to use
- The user asks to "sort the inbox" / "file my notes" / process new captures.
- After sweep-daily-notes, as part of the combined sort run.

## Before you start
1. Read `.claude/rules/taxonomy.md` (types + routing map) and `conventions.md` (names).
2. Fetch only Inbox records with `Status=New`.

## Step 0 — completion vs new
First decide whether the note marks an EXISTING item done/updated, rather than a new item.
Cues: "closed", "finished", "did", "done", "completed", or a clear reference to a known task/goal.

If it is a completion/update:
1. Search the relevant base (Tasks / Goals / Projects) for a match by meaning and title.
2. **Exactly one confident match** → mark it done (Task: `Done`=true; Goal/Project: `Status=Done`).
   Then:
   - write the change to the run audit log (what-changed.md),
   - enqueue an **Outbox** row `{Type: notify, Handler: Concierge Gateway, Payload: "Closed
     <base>/<title>", Status: Queued}` so the Gateway posts a Telegram log,
   - set Inbox `Target="closed: <base>/<title>"`, `Status=Sorted`.
3. **No match or several plausible** → **Not Recognized** (Reason: "completion note, no confident
   match"). Never close on a weak match.

Otherwise treat it as a new item and continue below.

## Step 1+ — classify & file new items
1. Assign a `Type`: task · reminder · reference · goal · thought · event · expense · purchase · unsure.
2. Confident → write to the destination base per the routing map and set its fields.
   - Reversible (note/idea/goal) — write silently.
   - Implies an external action (calendar event, doc/sheet edit, other third-party) — set
     Executor=Auto and **enqueue an Outbox row** with the right `Type` (calendar/doc/sheet/other),
     `Handler=Steward (MCP)`, and a `Payload`. The executor routine performs it via MCP. (Telegram
     messages instead use `Type=notify, Handler=Concierge Gateway`.)
3. event / expense / purchase: no destination yet — set `Type`, leave `Status=New`, note in report.
4. Unsure / doubtful → write to **Not Recognized** (Note + Reason + Source). Do not guess.
5. For each filed record set `Status=Sorted` and fill `Target` (where it went) for audit.

## Rules
- Don't split partially recognized: file it whole OR send it whole to Not Recognized with a reason.
- Never delete records — only change Status.
- The sort routine never executes outbound work — it only writes Outbox rows with a Handler.
- reference ≠ action. Be conservative on completion matching: doubt → Not Recognized.

## Output (run report)
Brief: processed N; new items filed by base (with breakdown); items CLOSED (list each);
Outbox rows enqueued (by Type/Handler); X to Not Recognized; what is waiting on a connector.
