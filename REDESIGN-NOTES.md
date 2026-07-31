# Notes for whoever (or whatever AI) is working on this next

This is a single self-contained `index.html` (no build step, no external JS
dependencies, no fonts loaded from anywhere — pure system fonts). It's a
from-scratch retirement planner: unlike a personal tracker prefilled with one
person's numbers, this one starts empty and walks *any* user through entering
their own age, income, accounts, and assumptions before showing a plan. It's
wrapped as an installable PWA (`manifest.json` + `sw.js`), with all data
staying in the browser's `localStorage` — nothing is ever sent to a server.

The owner wants continued **visual/UX polish**, not a change in what the tool
does or how the math works, unless asked. As of this note, the app is fully
functional end-to-end (data entry → validation → calculated plan with chart,
income breakdown, and milestone table) — a first round of real validation
bugs was just found and fixed (see "Bugs already fixed" below), so treat this
as a working baseline, not a draft to debug from scratch.

## Workflow reality check (read this first)

Don't assume you can open a branch or PR against this repo — the AI tool
reading this file typically has no way to push to git, only to hand back
edited file(s) for the owner to manually apply and redeploy. Make your edits
and clearly say which file(s) changed; don't promise a PR you can't open.

## What's safe to change (pure styling)

- All color values live as CSS custom properties in the `:root{...}` block
  near the top of `<style>`: `--page`, `--panel`, `--panel2`, `--ink`,
  `--muted`, `--rule`, `--green`, `--blue`, `--violet`, `--amber`, `--red`.
- The `COLORS` array near the top of `<script>` (8 hex values) — used for
  each account's chart line color and legend dot, cycled by index. Update
  alongside the CSS palette if you want the chart to match a new theme.
- Fonts, spacing, border-radius, card layout, button styles — all fair game.
- `icon.svg` and `manifest.json`'s colors/name — free to redesign, just keep
  the filename referenced in `manifest.json`, `index.html`'s `<head>`, and
  `sw.js`'s `ASSETS` array in sync if you rename anything.

## Do not change (functionality — breaking these breaks the tool)

- Element `id` attributes. The script finds everything by id (`$ = id =>
  document.getElementById(id)`) — renaming one without updating every `$("...")`
  call will silently stop that field from working.
- The `validate()` function's `isNum(...)` guards. A real bug was just fixed
  here: blank required fields (income, ages, account balance/contribution)
  used to silently pass validation as if they were `0`, because in
  JavaScript `null >= 0` and `null >= null` are both `true`. Every numeric
  check now requires `isNum(v)` (a real finite number) before the range
  comparison runs. Don't strip that guard back out for "simpler" code.
- The `localStorage` key (`retirement-planner-all:v1`) and the shape of the
  saved `state` object (`currentAge`, `retirementAge`, `income`,
  `lifeExpectancy`, `returnRate`, `inflation`, `withdrawalRate`,
  `contributionGrowth`, `pension`, `socialSecurity`, `otherIncome`,
  `accounts[]`). Changing field names without a migration will make
  existing users' saved plans silently disappear on their next visit.
- The account object shape (`id`, `name`, `type`, `balance`, `contribution`,
  `returnOverride`) and the `TYPES` map — `drawChart`/`project`/`renderResults`
  all key off these exact property names.
- The service worker's network-first fetch strategy (`sw.js`) — it always
  tries the network first and only falls back to cache when offline, which
  is the opposite of a cache-first strategy. That's deliberate for a tool
  whose whole point is a private local calculator; don't swap in a
  cache-first pattern without a reason, and if you add new files, add them
  to `ASSETS` and bump `CACHE` so installed phones actually pick up changes.

## If you want to change the chart's drawing style

`drawChart(p)` in `<script>` is pure hand-drawn SVG (`<polyline>` per
account, plus a bold total line) — no charting library, and it should stay
that way for the same offline-reliability reasons as the sibling project:
no CDN dependency for anything the tool needs to function. It redraws from
scratch on every `calculate` and on window resize; keep it doing that.

## Bugs already fixed (don't reintroduce)

- Blank-field validation silently passing (see `isNum` above).
- Multi-account forms gave no indication of *which* account was invalid —
  `.invalid-card` now highlights the specific offending account card in red
  when the accounts group fails validation. Keep that per-card check when
  touching `validate()`.

## Where this lives

Live site: [dandrews77.github.io/retirement-planner-all](https://dandrews77.github.io/retirement-planner-all/),
built from the public repo `DAndrews77/retirement-planner-all` on GitHub.
