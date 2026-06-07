---
name: bootstrap-notion
description: Creates the full Steward Notion structure from scratch — all bases with fields, relations, Area rows, filtered views, and emojis. Use once on first-time setup / fresh install, or when the user asks to bootstrap, initialize, or recreate the Notion bases. Requires the Notion connector with permission to create databases and pages in a chosen parent page.
---

# Bootstrap Notion

One-time setup. Creates every base the system needs, wired with relations. Schema reference:
`docs/data-model.md`; exact names/casing: `.claude/rules/conventions.md`.

## Prerequisites
- Notion connector connected, able to create databases/pages.
- Ask the user which **parent page or teamspace** to create everything under. Create a page
  named **Steward** there if they want a clean container.

## Order (relations need their targets to exist first)
Create each as a database under the parent. Put the emoji in the title (e.g. `📥 Inbox`).

1. **📥 Inbox** — `Note` (title), `Status` (select: New, Sorted), `Type` (select: task, reminder,
   reference, goal, thought, event, expense, purchase, unsure), `Target` (text). Date = created time.
2. **🔁 Areas DB** — `Area` (title), `Notes` (text). Then add rows:
   Career, Health, Sport, Money, Family, Content, Other.
3. **🎯 Goals** — `Goal` (title), `Horizon` (select: Yearly, Short-term), `Target date` (date),
   `Status` (select: Not started, In progress, Done), `Notes` (text),
   `Area` RELATION → Areas DB (two-way, synced name "Goals").
4. **🚀 Projects** — `Project` (title), `Status` (select: Backlog, Active, On hold, Done),
   `Deadline` (date), `Summary` (text), `Area` RELATION → Areas DB (two-way "Projects"),
   `Goal` RELATION → Goals (two-way "Projects").
5. **✅ Tasks** — `Task` (title), `Done` (checkbox), `Do date` (date),
   `Priority` (select: Top, Secondary), `Type` (select: Work, Personal),
   `Tag` (select: Triage, From Daily, Reminder), `Assignee` (text),
   `Executor` (select: Me, Auto (Steward)), `Project` RELATION → Projects (two-way "Tasks").
   Then add two table views: **Work** (filter Type=Work), **Personal** (filter Type=Personal).
6. **💡 Content Ideas** — `Idea` (title), `Platform` (select: Telegram, X, LinkedIn),
   `Status` (select: New, In progress, Drafted, Posted), `Notes` (text),
   `Area` RELATION → Areas DB (two-way "Content Ideas"; optional to fill).
7. **🗒️ Reviews** — `Period` (title), `Date` (date), and text columns:
   Health, Sport, Career, Work, Money, Family. (Wide format: one row = one period.)
8. **⚠️ Not Recognized** — `Note` (title), `Reason` (text),
   `Status` (select: Open, Resolved), `Source` (select: Inbox voice, Daily notes).
9. **📡 Outbox** — `Item` (title), `Type` (select: notify, calendar, doc, sheet, other),
   `Handler` (select: Steward (MCP), Concierge Gateway), `Payload` (text),
   `Status` (select: Queued, Done, Failed), `Source` (text).

## After
- Report each base created with its **database ID** (the user pastes the Inbox ID into the iOS
  shortcut and any IDs the Concierge Gateway repo needs).
- Remind the user to connect their capture integration to **Inbox** (••• → Connections) so the
  shortcut can POST to it.
- Leave all bases empty except the Area rows. Never insert personal data here.

## Rules
- Relations are two-way (DUAL); the back-relation appears automatically on the target base.
- Use exact names and casing from `conventions.md`. Do not invent extra fields.
- If a base already exists, update it to match rather than creating a duplicate.
