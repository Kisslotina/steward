# Notion call forms (write contract)

Copy these parameter shapes verbatim — never improvise them. Each one is a shape that
previously failed and cost a retry. Read this file once at the start of a run that will
write rows (Step 3), and reuse it for every write.

1. **`notion-fetch` — always by `id`, never `url`:**
   `notion-fetch { id: "<page_id>" }`.
   Passing `url:` makes every fetch in the batch fail.

2. **`notion-move-pages` — param is `page_or_database_ids`, ONE batched call (≤100 ids):**
   `notion-move-pages { page_or_database_ids: [<ids…>], new_parent: { type: "data_source_id",
   data_source_id: "<Inbox Archive id>" } }`.
   Not `page_ids`.

3. **`notion-create-pages` — `parent` at the TOP level, not inside `pages[]`; dates expanded:**
   `notion-create-pages { parent: { type: "data_source_id", data_source_id: "<dest id>" },
   pages: [ { properties: { "<title-field>": "…", "date:<Field>:start": "YYYY-MM-DD", … } } ] }`.
   A `parent` key inside a `pages[]` object errors "Unrecognized key: parent".
   Write dates as `date:<Field>:start` (e.g. `date:Do date:start`) — a bare `Do date`
   errors "Date type must be expanded".

4. **Area relation — always the full Notion URL** from the cached `areas` map (Step 0),
   never a bare page id (a bare id errors "Invalid agent URL").

5. **Select properties — only values from the write-contract** (`selects` in
   `schema.cache.json`). Anything else errors "Invalid select value" (e.g. free-text
   "December 2026" into Goals.`Horizon`, whose only values are `Yearly` / `Short-term`).

6. **Reviews has no append** — read-modify-write:
   `notion-fetch` the period row → read the area column → `notion-update-page` with the
   full new value (old text + newline + new line). There is no `append` command.
