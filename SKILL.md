---
name: chaos-day-tracker
description: Multi-thread day tracker for people who run several AI chats at once on different work. Keeps one shared page of what got done, what shipped, what's outstanding, and what needs a decision. ALWAYS use this when the user says chaos, chaos day, chaos mode, chaos page, "keep track of what I'm doing today", "I have X chats open", "update the page with what we did here", or pastes a cross-thread status-update prompt. Also use it automatically at the start of a new work session when you can see that several of the user's own chats have been active in the last few hours. Do NOT trigger for automated agents, scheduled tasks, or background runs — only for chats the user is typing into personally.
---

# Chaos Day Tracker

High-output days are the most productive and the most lossy. You run several AI chats at once, each one produces real work, and by evening the state of that work lives in five places and nowhere. This skill makes one page the state.

It is a tracker, not a nag. It never hands the user homework, never asks a question, and never interrupts the work in the chat it runs in.

## Config (edit to taste)

```
THREAD_THRESHOLD  = 4        # user-typed chats active at once before chaos mode fires
ACTIVE_WINDOW     = 6 hours  # how recently a chat counts as "active"
PAGE_NAME         = Chaos_Page_YYYY-MM-DD   (user's local date)
PAGE_LOCATION     = wherever the AI can write: Google Doc, Notion page, Obsidian note,
                    a text file, or — if it can't write anywhere — the chat itself
REPROMPT_OFFSET   = +12 hours, clamped into 07:00–21:00 local time
```

## Step 1 — The tripwire

Run this check once per chat, on the first substantive turn of a new piece of work, or any time the user references other chats:

1. If you can see the user's recent chats, count the distinct ones — including this one — updated inside ACTIVE_WINDOW that are working on **different things**.
2. If the count ≥ THREAD_THRESHOLD, chaos mode is on. Otherwise say nothing and work normally.
3. If you **cannot** see other chats, chaos mode is on only when the user says so ("chaos mode", "chaos day", "I have 5 chats open", or pastes the close-out prompt).

Only user-typed chats count. Always exclude:
- scheduled tasks and any chat whose recent turns are automated output
- agent runs, coding-agent sessions, sub-agents, anything the AI spawned
- chats on the same subject as this one (that's one work stream, not two)

If it's ambiguous whether a chat is user-typed, don't count it. Failing to trigger costs one page entry; false-triggering on background automation makes the skill noise and the user will turn it off.

Never announce the check. If chaos mode fires, the only thing the user sees is one line at the end of the reply: **Chaos page updated — [link or location].**

## Step 2 — Find or create today's page

One page per chaos day, shared by every chat.

- Search for a page titled `Chaos_Page_<today>`. If it exists, use it — never create a second.
- If the user linked or named a page themselves this session, that page wins. Use it; do not create a parallel one.
- If neither exists, create `Chaos_Page_<today>` from `TEMPLATE.md`.
- If you have no way to write to a page, output the entry in chat in the exact format below so the user can paste it, and say plainly that you can't write to a page.

A chaos day ends at close-out, not at midnight. A chat still running at 1 AM writes to the day it started.

## Step 3 — Read before you write

Read the whole page first. Then, in your entry, flag any **collision**: another chat touching the same file, page, doc, repo, campaign, or asset. Two chats editing one artifact from different assumptions is the most expensive failure of a chaos day, and it is invisible from inside either chat.

## Step 4 — Write the entry

Append at the end of the page:

```
THREAD: <short name> — updated <Mon D, ~H:MM AM/PM>

DONE (last 24h)
- [LIVE]  <thing that is deployed, sent, published, or in someone's hands>
- [BUILT] <thing that exists but is not live anywhere>

OUTSTANDING
1. <item> — <why it's not done: blocked / needs a specific device or login / awaiting decision / just next>

COLLISIONS
- <other thread name> is also touching <artifact>   (or: none)

DISPOSITION: CLOSE  (durable state written, nothing left here)
        or   KEEP OPEN — <the one specific reason>
```

Then update the three standing blocks at the top of the page:

- **DECISIONS NEEDED** — one line each, phrased so a yes/no or A/B answer unblocks it. Decisions go at the top of the page, never trailing in an entry.
- **SPEND + IRREVERSIBLE TODAY** — credits burned, money spent, sends to real lists, deploys to live sites, deletions. On a high-output day an expensive mistake hides inside a good one.
- **DEVICE-BOUND PUNCH LIST** — anything that needs a specific machine, browser profile, or login, grouped so the user does them in one sitting instead of hitting the same wall in five chats.

Every [BUILT] item is a question the day has not answered. At close-out the LIVE/BUILT ratio is the honest score of the day.

## Step 5 — Verify the write

After any write, re-read the page and confirm a unique phrase from what you just wrote is there. Unverified writes fail silently. If the write fails twice, post the entry in chat in the exact format above so the user can paste it, and say plainly that the write failed.

## Step 6 — The re-prompt

When chaos mode first fires on a given day, set a reminder (calendar event if you can create one; otherwise tell the user to set one) at REPROMPT_OFFSET titled **"CHAOS CLOSE-OUT — sweep open chats"**, with this in the body so it works without any single chat:

> Paste into every open AI chat: "Chaos close-out. Read the chaos page, add what this chat did since its last entry, mark DONE items LIVE or BUILT, list what's outstanding, and write anything durable to wherever it permanently lives. Then tell me if this chat can be closed."

An AI cannot send a message into another chat. The reminder is the delivery mechanism. Say that in one clause, not a paragraph.

## Step 7 — Close-out

On the close-out run, in addition to the normal entry:

1. Sweep the page. Push every durable item to wherever it permanently lives (project doc, notes system, task tracker). The chaos page is never the system of record.
2. Report the day in four lines: shipped count, built-not-shipped count, decisions still open, chats safe to close.
3. Name the chats to close. Default is close. Once durable state is written and verified, an open chat is a cost, not an option.

## Hard rules

- **Never ask a question.** Pick the most likely option, state the pick in one line, proceed.
- **Never end a reply with a chore.** Decisions go at the top of the page; open items live on the page, not in the user's face. A trailing ask reads as unfinished work.
- **Never let the tracker eat the work.** One check, one read, one write, one verify. If tracking costs more than a minute of the chat's attention, it's built wrong.
- **Silent by default.** The page speaks when something is done, shipped, blocked, or colliding. It is not a transcript of the session.
- **Automation never counts.** This fires on the user's own typing only.
