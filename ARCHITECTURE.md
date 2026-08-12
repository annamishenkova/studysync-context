# StudySync — architecture

## Stack

- **React 19 + TanStack Start v1** (Vite 7). File-based routing in `src/routes`.
  `src/routes/__root.tsx` is the only root layout; it imports `../styles.css` directly.
- **Tailwind CSS v4** via `src/styles.css` (`@theme` tokens, no `tailwind.config.js`).
- **Supabase** for database, auth, and realtime presence.
- **PWA** service worker (`vite-plugin-pwa`) for offline modules.

## Layout

```text
src/
  routes/            file-based routes; each thin route re-exports a page
  pages/             the actual screens (Timer, Dashboard, Tasks, Coworking, ...)
  components/        grouped by area: timer/, dashboard/, tasks/, coworking/, subject/, ...
  context/
    TimerContext.jsx Focus Guard state machine — the heart of the app
  lib/
    focusGuard.js    settings, persistence, noise window
    allowedTabs.js   app-launched link tracking + domain allowlist
    taskBoard.js     board CRUD, AI triage, minute crediting
    classLinks.js    course auto-join scheduling
    offlineQueue.js  queued writes while offline
    stats.functions.ts  verified-session aggregates for landing evidence
    ai.functions.ts / ai.server.ts  gateway calls, shared default model
  routes/api/
    ai-stream.ts     streaming interpreter
    youtube.ts       credit-free YouTube search + captions
  integrations/supabase/  generated clients (never edit)
```

## Server boundaries

- App-internal logic uses `createServerFn` from `@tanstack/react-start`, declared in
  `*.functions.ts` wrapper modules; runtime helpers live in imported `*.server.ts` files.
- Raw HTTP endpoints (webhooks, public APIs) live under `src/routes/api/`; anything a
  third party calls goes under `src/routes/api/public/*` and verifies the caller itself.
- `process.env` is read inside handlers only. Browser config uses `import.meta.env.VITE_*`.

## Focus Guard state machine (`TimerContext.jsx`)

1. Tick a client-side clock; persist state to `localStorage` so a crash is recoverable.
2. On `visibilitychange` / `blur`, start an away span unless physical mode is on.
3. If the away target is an allowlisted host the app just launched, credit it as allowed.
4. After the grace window: coach/honest modes pause and rewind to the departure second;
   strict mode stops and raises an interrupt prompt for the user to resolve.
5. On return, spans shorter than the noise window are ignored; longer ones add away time
   and a departure, plus an "was that study?" chip to reclaim them once.
6. On save, compute `focused_seconds`, `away_seconds`, `duration_minutes`,
   `distraction_count`, `interrupted`, `focus_guard_verified`, then emit a receipt and
   credit honest minutes to the active task card.

## Database notes

- Every public table has explicit `GRANT`s plus RLS policies; role checks go through a
  `security definer` helper, never a column on a profile row.
- Coworking RPCs are restricted to authenticated users.
- `subject.hard_deadline` powers countdown and overdue warnings on the dashboard.

## Conventions

- Never edit `src/routeTree.gen.ts` or generated Supabase files.
- Guest mode uses deterministic demo data (`src/api/demoData.js`) and migrates into a real
  account on sign-in (`src/api/demoMigrate.js`).
- Panels in Subject Lab are wrapped in `PanelErrorBoundary` so one crash cannot take the
  page down.
