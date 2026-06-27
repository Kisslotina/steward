# Recovery paths (rare — read only when triggered)

These paths fire in the minority of runs. The normal hot path never needs them; trust the
invariants in SKILL.md unless one of the specific triggers below holds.

## 1. Self-heal a missing or stale schema cache

Trigger: `schema.cache.json` is absent, OR a write errors because the cache disagrees with
Notion — "Property … not found", "Date type must be expanded", "Invalid select value",
"Invalid agent URL".

Action: discover the affected base's schema **once** via `notion-fetch`, write the corrected
write-contract back to `schema.cache.json`, then keep reading from the file.
Discover **at most once per base per run** — never per item. The Areas map is permanent
(Areas DB is canonical and effectively never changes) — never re-fetch Area URLs.

## 2. Leftover guard (Sorted rows lingering in the live Inbox)

Trigger: `inbox.lastArchiveFailed = true` in `schema.cache.json` — set by a prior run whose
Step 4 archive move failed. **Only when this flag is set.**

Action: run a one-time per-page `notion-fetch` over the current batch and drop rows already
`Status=Sorted` — send them to Step 4 archive, never re-file. Then clear the flag.
When the flag is unset (the normal case) trust the invariant and do not fetch.

## 3. Archive-move outcome recording

After the Step 4 batched `notion-move-pages`:
- On success → set `inbox.lastArchiveFailed = false` in `schema.cache.json`.
- If the move still fails after its one allowed retry → set `inbox.lastArchiveFailed = true`
  (so the next run runs the one-time reconciliation in §2) and report the failure.

If `Inbox Archive` is absent from the registry, skip the move, leave the rows `Sorted`
in place, and note it in the report (run `bootstrap-notion` to add the archive). A `Sorted`
row left in the Inbox is still correct — §2 sends it straight to archive next run.

## 4. Transient server errors on a write

Trigger: a 5xx / timeout / "service unavailable" on a single write — usually transient.

Action: retry that one call **exactly once**. If it fails again, leave that Inbox row `New`
and report it — do not abort the batch, do not retry more than once. This applies ONLY to
transient server errors, never to the validation errors in `notion-call-forms.md` (those are
contract bugs — fix the call shape or self-heal the cache per §1, do not retry blindly).
