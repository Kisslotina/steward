# Profile — context template

> **What this is.** The personal context the assistant needs to work well for *you*.
> Fork this project, copy this file to `docs/profile.md`, and fill in the answers.
> `docs/profile.md` is git-ignored — it stays on your machine and is never published.
>
> **Why it exists.** The agent (`Steward`) holds *structure and rules*, not your biography.
> Your personal facts live here, separate from the logic. This file later seeds the assistant's
> ongoing `life-context` (the file the future advisor agent reads). Keep it factual and short.

---

## 1. Who you are
- Name / how the assistant should address you:
- Age / life stage:
- Location & time zone:
- Languages (for capture vs. for replies):
- Household (partner, kids, anyone the assistant should know about):

## 2. Work
- Role / company:
- What your work mainly involves:
- What "a good work week" looks like:

## 3. Life areas (your Areas DB)
List the life domains you actually want to track, and one line on what each means to you.
(Default canon: Career · Health · Sport · Money · Family · Content · Other — edit to fit.)
-
-

## 4. Goals & horizon
- Current yearly goals:
- Shorter-term goals (this quarter / next weeks):
- How often you review (e.g. every two weeks + monthly):

## 5. Capture habits
- What you tend to capture by voice (tasks, expenses, ideas, …):
- Where work notes go (e.g. Daily notes on the `Today` page):
- Anything that should always be treated a certain way:

## 6. Working style & tone
- How you want the assistant to communicate (concise? detailed? formal?):
- Things it should never do:
- What "done well" means to you for the sort routine (your readiness bar):

## 7. Tools & accounts (no secrets here)
- Notion workspace / teamspace name:
- Capture device (e.g. iPhone 16 Pro Max — Hey Siri / Action Button / Back Tap):
- Messaging channel for confirmations (e.g. shared Telegram):
- Calendar (e.g. shared Google Calendar):

> Put actual API keys / tokens only in `.env` (see `.env.example`), never in this file.
