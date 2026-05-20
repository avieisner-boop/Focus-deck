# Focus Deck — Handoff Notes

> Drop this file into the `avieisner-boop/Focus-deck` repo so the next Claude session has full context. Then paste the **Session prompt** below into a new Claude Code session pointed at that repo.

---

## Who this is for

**Avi** (avieisner@gmail.com) — building a daily command center for an ADHD brain that:
- struggles with organization and follow-through
- thinks in 12 directions at once
- needs low-friction capture and visible reminders
- uses many remote tools (Gmail, Calendar, Drive, Notion, Airtable, QuickBooks, Shopify, WordPress, Slack, etc.)
- works on iPhone and laptop, wants the same data on both

## What Focus Deck is

A self-contained single-file web app — **HTML + inline CSS + inline JS**. No backend, no build step, no dependencies. Stored in the browser's `localStorage`. Designed to "just work" when the file is opened in any browser.

**Live URL:** https://avieisner-boop.github.io/Focus-deck/focusdeck.html
**Repo:** https://github.com/avieisner-boop/Focus-deck (public, GitHub Pages enabled, Deploy from branch: `main`, root)
**Source file:** `focusdeck.html` (formerly `Focusdeck.html` — uppercase version may still exist; safe to delete)

## Features shipped (v2)

### Capture
- One text bar at the top
- Inline shorthand parses metadata as you type:
  - `#tag` adds tags
  - `@project` assigns project
  - `!high` `!med` `!low` priority
  - `^today` `^tomorrow` `^fri` `^2026-06-01` `^6/12` due date
  - `~low` `~med` `~high` energy required

### Views
- **Today** — overdue + due today + items with no date + focused items
- **Upcoming** — future-dated
- **Inbox** — unsorted (no date, no project)
- **Routines** — recurring (daily/weekdays/weekly/monthly), auto-respawn on completion
- **Someday** — parked ideas
- **Done log** — completed items, dopamine receipts
- Project views (sidebar)
- Tag views (sidebar chips)

### Task cards
Click ▾ to expand. Each card supports:
- Inline-editable title
- Notes (free text)
- Due date / priority / energy level / project / tags / repeat
- **Attachments**: link (paste URL) or file upload (data URL in localStorage; warns >4MB)
- Unlimited subtasks
- "Copy share text" button — formats for Slack/email
- Snooze (defers 1 day)
- Move to Someday / back to active

### ADHD-targeted features
- **Energy check-in** (🪫 / ⚡ / 🔥) gives a smart suggestion for what to tackle
- **Streak counter** — daily completion streak
- **Focus timer** — Pomodoro (25/15/50 min) with body-double "pick one task" flow, browser notification at end
- **Mini calendar** — visual sense of when deadlines land
- **Search** (`/` to focus)
- **Keyboard shortcuts**: `/` search, `n` new task, `f` toggle focus
- **Hide completed by default** (toggle in Settings)
- **Light/dark themes**

### Quick links (right rail)
- **Daily Rundown** — Gmail, Outlook, Google Chat, Slack, WhatsApp, Messenger, Teams, Discord, Telegram, iMessage (iCloud)
- **Remote Tools** — Google Calendar/Drive/Docs/Sheets, Notion, Airtable, Dropbox, OneDrive, QuickBooks, Shopify, Canva, WordPress, Vercel, GitHub, Claude, ChatGPT, Gemini, Zendesk, RingCentral, SendGrid, Twilio
- **Custom links** — user-defined

### Data
- **Export** writes a JSON backup
- **Import** restores from JSON

## What's NOT done yet (priorities for next session)

### 1. Cross-device sync (TOP PRIORITY)
Avi has multiple devices (iPhone + laptop). Current state: each device has its own `localStorage` — no sync. Manual export/import to iCloud Drive works but is friction.

**Recommended approach: GitHub Gist auto-sync (Option B from chat)**
- User creates a Personal Access Token with `gist` scope
- Paste token into Settings → saved encrypted in localStorage
- App polls / pushes to a single private gist on:
  - Page load
  - After every change (debounced ~5 sec)
  - On window focus
  - Manual "Sync now" button
- Conflict strategy: last-write-wins with a "last synced" timestamp shown in UI
- ~60-100 lines of code added to existing single file

**Alt (more effort, better UX):** Supabase free tier — real-time sync, files in cloud not localStorage, requires sign-in flow.

### 2. PWA polish
- Already works via "Add to Home Screen"
- Could add a proper manifest + service worker for offline + nicer icon
- Icon currently inlined as SVG favicon

### 3. Smarter today view
- Suggest task to start based on energy check-in + priority + due date
- "Next action" card pinned at top

### 4. Better mobile expand UX
- Tap-anywhere-on-card to expand (currently tap the ▾ button)
- Swipe gestures: swipe right to complete, left to snooze

### 5. Recurring task improvements
- Custom intervals (every 3 days, every 2nd Tuesday)
- Skip without breaking the streak

### 6. Reminders via push notifications
- Currently: in-browser Notification API only fires while a tab is open
- For real reminders: requires push API + service worker + ideally a backend
- Workaround: lean on Google Calendar links for time-based reminders

## Tech notes

- Single file: `focusdeck.html` — all CSS, all JS inline
- No build step, no npm
- Storage key: `focusdeck.v1` in `localStorage`
- Theme persists across sessions
- File attachments stored as base64 data URLs → eats localStorage fast for big files
- Total file ~1400 lines; readable, commented at section breaks
- All external links open in new tab with `noopener`
- No analytics, no telemetry, no network calls except user-clicked links

## Deployment

- Push to `main` branch → GitHub Pages rebuilds → live in ~30 sec
- No CI required
- Cleaner URL goal: rename `focusdeck.html` → `index.html` so URL becomes `https://avieisner-boop.github.io/Focus-deck/`

## Decisions made along the way

- **Why a single HTML file?** Frictionless install. Avi has ADHD; any build step kills adoption.
- **Why localStorage?** Same reason. Sync is a v2 problem.
- **Why GitHub Pages?** Free, public repo, auto-deploys on push. No signup beyond GitHub.
- **Why not Notion / Todoist / TickTick?** Avi wanted *his* tool with *his* energy check-in and *his* remote-tool launcher; existing apps don't combine these.
- **Capture syntax vs forms?** Syntax is faster for ADHD brain dumps. Forms slow you down.
- **Today view includes no-date tasks** (changed in v2 mobile fix) — because users expect "I just typed it, it should be visible."

## Files in repo

```
Focus-deck/
├── README.md           # public-facing description
├── HANDOFF.md          # this file
├── focusdeck.html      # the app (v2, mobile-fixed)
└── Focusdeck.html      # OLD v1, safe to delete
```

---

## Session prompt for the next Claude session

Copy-paste this into a new Claude Code session opened on the `avieisner-boop/Focus-deck` repo:

```
I'm continuing work on Focus Deck — an ADHD-friendly daily command center I
built in an earlier Claude session. The app is a single HTML file
(focusdeck.html) at the repo root, deployed live via GitHub Pages at
https://avieisner-boop.github.io/Focus-deck/focusdeck.html.

Please start by reading HANDOFF.md — it captures what's built, decisions
made, and what to tackle next.

Top priority: add automatic cross-device sync via GitHub Gist (Option B from
the handoff). I have ADHD, so:
- Make the setup walkthrough step-by-step with screenshots/clear directions
- Default to "just work" once a token is pasted
- Don't ask me questions you can answer yourself

I'm on iPhone most of the time. When you push changes, they go live on the
URL above automatically. Please commit small, descriptive changes so I can
see what shipped.

Also feel free to delete the old Focusdeck.html (capital F) if it still
exists — it's the v1 with the mobile bug.
```

## Open file in the new session

Point the session at: **`focusdeck.html`** (root of the `Focus-deck` repo).
This is the entire app. The HANDOFF.md sits alongside it for context.
