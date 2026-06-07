---
name: sort-inbox
description: Classifies and files raw Inbox records into the EXISTING typed Notion bases, applies completion notes that close existing items, and enqueues outbound work to the Outbox. Use after capturing notes, or whenever the user asks to sort/process the Inbox. Runs only after sweep-daily-notes. Reads Inbox rows with Status=New, decides new-vs-completion, routes to a real base, sets Status=Sorted and Target.
---

# Sort Inbox

Processes every Inbox record with `Status=New`. Runs ONLY after `sweep-daily-notes` completes.

## Step 0 — GROUND YOURSELF IN THE REAL BASES (do this first, every run)
Before classifying anything:
1. Query Notion for the **actual list of databases** available via the connector. Record each
   base's **exact title and data-source ID**.
2. Build the routing map against THESE real bases only (match by exact name from
   `.claude/rules/conventions.md`). 
3. **NEVER invent, rename, abbreviate, or guess a base** (no "Reflections DB", "Shopping DB",
   "Ideas DB"). If a taxonomy destination has **no matching real base**, those items go to
   **Not Recognized** with reason "no destination base: <type>" — do not fabricate one.

## Step 1 — fetch work
Fetch only Inbox records with `Status=New`.

## Step 2 — completion vs new
If a note marks an EXISTING item done (cues: "closed", "finished", "done", names a known task/goal):
- find one confident match in a real base → set it Done; write to the audit log; enqueue an Outbox
  `{Type:notify, Handler:Concierge Gateway, Status:Queued}` row; set Inbox `Target="closed: …"`.
- no/ambiguous match → Not Recognized. Never close on a weak match.

## Step 3 — classify & FILE new items (actually write the row)
1. Assign a `Type` from the taxonomy (`.claude/rules/taxonomy.md`).
2. Map it to a REAL base from Step 0. If none exists → Not Recognized.
3. **Create the actual page/row** in that base's data source with the proper fields (use the
   data-source ID from Step 0). Do not just describe it — perform the write.
4. **Verify** the new row exists (it has a URL/ID), then set Inbox `Status=Sorted` and
   `Target="<RealBaseName>/<title>"`. If the write failed, leave Inbox `New` and report the error —
   never report success for a write that didn't happen.
5. External action → enqueue an Outbox row (Handler per taxonomy) instead of acting directly.

## Rules
- Only route to bases returned by the connector in Step 0. No invented names.
- Don't split partially recognized: file whole OR Not Recognized with a reason.
- Never delete records — only change Status. The sort routine never executes outbound work.
- reference ≠ action. Doubt → Not Recognized.

## Output (run report)
Per item: input → Type → REAL base it was written to (with confirmation) OR Not Recognized (reason).
Totals: filed, closed, enqueued to Outbox, Not Recognized. Flag any write that failed.
