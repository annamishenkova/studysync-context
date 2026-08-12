# StudySync — project context

Single source of truth for what this app is, who it's for, and the rules it must respect.
Written so another human or AI can pick the project up without re-deriving intent.

## Purpose

StudySync is a real daily study tool for university, not a demo. Everything must be
plug-and-play: usable on day one, no setup theatre, no fake data in the user's own view.

The differentiator is **Honest Hours**: the difference between the time a timer says you
studied and the time the app could actually verify. Focus Guard is the mechanism, and it
stays central on every surface (landing evidence, dashboard, timer, receipts, coworking).

## Core product rules

- A timer left running is not study time. Anything unmeasurable is labelled unverified.
- Sessions shorter than 2 minutes are noise and are discarded, never logged.
- The user decides what an interruption meant (continue / end and log / discard).
  The app never silently ends a session behind their back.
- Away time on an allowlisted resource the app opened is credited, not punished.
- Self-declared away time may be credited by hand, but the session then loses `verified`.
- Physical / off-screen work is credited at a fixed 75% focus estimate and is never verified.
- Honest minutes credited to a session flow back to the task-board card it was started from.

## Removed on purpose — do not re-add

- The "Life" group (finances, reading, habits).
- Concepts / Articles / Code Library modules.

## Design rules

- Dark-first theme driven entirely by semantic tokens in `src/styles.css`. No hardcoded
  colour utilities (`text-white`, `bg-black`, `bg-[#...]`) in components.
- Dimmed borders everywhere; no bright white checkbox or pill borders.
- Hover brightens the **border**, not the background.
- Big timer uses monospace, semibold, tabular numerals.

## Feature map

| Area | What it covers |
| --- | --- |
| Timer | Focus Guard session, physical mode, retro logger, session receipt |
| Dashboard | Tracked week, focus quality, streaks, plan generator, deadlines |
| Tasks | Smart Kanban board, homework sync, focus-gated cards |
| Coworking | Rooms with QR/code invites, presence heartbeat, shared honest minutes |
| Subject Lab | Interpreter (streaming), quizzes with LaTeX, analogies, formula lab, YouTube |
| School / Courses | Timetable, homework, study log, auto-join class links 15 min before |
| Flashcards | Decks, study sessions, streaks, offline queue |
| Languages | Dictionary, grammar, exercises, essay panel, radar of weak areas |
| Exam War Room | Strategic dashboard and exam simulator |

## AI usage

All model calls go through the Lovable AI Gateway with `google/gemini-3.1-flash-lite`
(cheapest tier) as the shared default. YouTube search and transcripts cost **zero**
credits — they read YouTube's public pages server-side instead of asking a model.

## Offline

Timer, flashcards and homework work offline: reads come from cached lists, writes go to an
offline queue and flush on reconnect. The service worker is guarded so a stale build cannot
trap the user — dynamic-import failures force a reload.

## Data honesty invariants

- Every saved session carries `duration_minutes`; missing it means zero logged time.
- Landing-page evidence aggregates **verified sessions only**.
- Interrupted sessions are flagged and rendered in break colours in the calendar.
