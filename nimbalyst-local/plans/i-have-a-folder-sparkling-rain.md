# Plan: Rebuild Openweek docs from the new `/design` v2 "paper" export

## Context

The repo is effectively **greenfield** — `git ls-files` shows only a clean Nuxt scaffold
(`app/app.vue`, `nuxt.config.ts`, `package.json`); there is no `app/components/`, no `server/`.
The git log confirms it: `85f2490 Reset to clean scaffold for v2 rebuild`. So the entire `/docs`
set and `CLAUDE.md` are an **aspirational spec** (they claim "Phases 1–6 implemented"), not a
reflection of code.

The user has dropped a new design into `/design` (`Openweek weekly planner.zip` → the canonical
file is **`Openweek v2 (paper).dc.html`**) and wants the documentation/flow rebuilt from it. The
problem: the current docs still carry **v1 decisions** that the new design directly contradicts —
most visibly `docs/design.md`, whose body still specifies Inter + Caveat fonts, `#FAF7F0`, and a
DaisyUI OKLCH theme, even though its own header note admits "v2 supersedes". Other docs
(`data-model.md` especially) are half-reconciled: header notes mention new fields but the tables
omit them.

**Goal:** make the new design the single source of truth and bring the doc set back into internal
consistency.

### Decisions locked with the user (this session)
1. **Scope = full reconcile** — rewrite `design.md` + update every doc the design contradicts.
2. **Old docs updated in place** (no archive folder; git history carries the evolution).
3. **Fonts = self-host + keep the selector** — IBM Plex Mono/Sans plus the 4 font themes, but via
   `@fontsource` (not Google CDN), preserving the offline/private self-host goal.
4. **New concepts documented fully now** — task `note`, the task↔calendar-event link badge, and the
   `showCalendarEvents` user setting all land in the data-model/design docs in this pass.

## Ground truth extracted from `Openweek v2 (paper).dc.html`

**Layout (1480×940 frame):** top bar → 7-equal-column week grid → bottom list drawer. **No sidebar.**

- **Top bar:** brand square (accent) + `openweek` wordmark · week nav `‹ June 22 – 28 ›` + `Today`
  pill + `WEEK 26` · right cluster: calendars dropdown (3 colored dots + "3 calendars" ▾),
  progress text "X of Y done", search icon `⌕`, account avatar.
- **Week grid:** 7 columns, hairline `#EDEBE4` rules; **today** column tinted `accent @10%` with the
  date in a filled accent circle. Per-day header = date + 3-letter weekday. Empty "Write a task"
  ghost row with a blinking accent caret on today.
- **Task row:** toggle mark `○`/`✓`; text with optional **highlighter** (4 colors); rolled-over
  prefix `↪`; meta line `◷ time` · `↻ {wkdays|weekly|monthly|daily}` · `☑ done/total` subtasks;
  **calendar-link pill** `⤺ from {source}`; italic **note** line.
- **Imported calendar event row:** distinct boxed style (colored left border + tinted bg keyed to
  source), `◷ time` + source dot/name (`GCal`/`CalDAV`/`iCal`), `＋ task` convert button; has an
  active/selected state.
- **Convert-event popover:** event summary, `Keep linked to calendar event` checkbox, `＋ Make task`
  / `Cancel`.
- **Bottom list drawer:** active list name + item count, items in a 2-row column-flow grid, `＋ Add`;
  tab strip with per-list colored dot + name + count, active tab underlined in accent, `＋ New list`.
  Example lists: Someday, Work, Personal, Groceries, Reading (each colored).

**Theme tokens (exact):** page `#F2F1EC` · grid surface `#FFFFFF` · drawer `#FCFBF7` · ink `#2A2A28`
· hairline `#EDEBE4` · muted text `#8C887D`/`#AEA99D` · selection `#EAD9A0`.
**Accents (4):** butter `#EAD9A0` · mint `#CFE0CB` · sky `#CBDDE9` (**default**) · rose `#E7CDD4`.
**Highlighter colors:** butter `#EAD9A0` · mint `#D2E2CD` · sky `#CFDEEA` · rose `#E9D2D8`.
**Highlighter styles (per-user):** `Underline` (42% bottom gradient) · `Swipe` (full background).
**Calendar source colors:** GCal `#86B08B` · CalDAV `#9CBBD6` · iCal `#D3B488`.

**Configurable props (→ become per-user settings):**
| Prop | Options | Default |
|---|---|---|
| `fontStyle` | Plex Mono · Editorial (Newsreader) · Grotesk (Space Grotesk) · Typewriter (Spline Sans Mono) | Plex Mono |
| `accentColor` | butter/mint/sky/rose | sky `#CBDDE9` |
| `tagStyle` | Underline · Swipe | Underline |
| `weekStartsOn` | Monday · Sunday | Monday |
| `showCalendarEvents` | on/off | on |

## Changes by file

### 1. `docs/design.md` — full rewrite (primary deliverable)
Replace the v1 body entirely. New structure:
- **Aesthetic principles** — keep the "paper not dashboard" framing; update the palette to the exact
  hex tokens above (warm paper, hairline rules, muted-pastel highlighters).
- **Theme tokens** — document the exact light palette + the 4 accents + 4 highlighter colors as the
  canonical values. Keep the existing DaisyUI-is-plumbing stance; note the grid/drawer are custom and
  re-express tokens in the project's CSS-first Tailwind v4 + DaisyUI form (OKLCH conversions are an
  implementation detail, hex values from the design are the spec).
- **Typography** — IBM Plex Mono (display) + IBM Plex Sans (body); document the **4-way `fontStyle`
  selector** (Plex Mono / Editorial-Newsreader / Grotesk-Space Grotesk / Typewriter-Spline Mono),
  **self-hosted via `@fontsource`** (supersedes Inter + Caveat; call out the change explicitly).
- **Anatomy / component flow** — new section documenting each surface and its states: top bar,
  day column + today state, task row (+ all meta affordances: time, recurrence, subtasks, rollover,
  note, calendar-link), imported-event row + active state, convert-event popover, list drawer + tabs.
  Use a Mermaid flow for the convert-event-to-task interaction.
- **Highlighter styles** — Underline vs Swipe, as a per-user `tagStyle`.
- Keep & lightly update the existing **Accessibility**, **Responsive/mobile**, **Offline/PWA**
  sections (they still hold; reconcile the font/self-host line with the new self-host decision).

### 2. `docs/data-model.md` — reconcile tables with the design's settings + fields
- **User `additionalFields` table:** add the rows that today only live in the stale header note —
  `accentColor`, `tagStyle`, `showCalendarEvents`, **and the new `fontStyle`** (text, default
  `'plex-mono'`). Keep `timezone`/`weekStartsOn`/`rolloverEnabled`/`lastRolloverDate`.
- **`tasks` table:** the table body is missing fields the header already promises — add `startTime`
  (`'HH:mm'` label) and `linkedEventId` (uuid? → `synced_events`, backs the `⤺ from {source}` badge
  and the "keep linked" convert option). `notes` already present — keep.
- Refresh the **Status** header note so it matches the tables (remove the "added since" drift).

### 3. `docs/decisions.md` — record the new one-way doors
Append decisions for: self-host fonts **with** a 4-theme selector (supersedes Inter+Caveat);
exact paper palette + 4 accents/highlighters as locked tokens; default accent = **sky**; `fontStyle`
+ `showCalendarEvents` as per-user settings; `tasks.linkedEventId` as the convert-to-task link.
Reference the prior D-numbers and continue the sequence.

### 4. `docs/tech-stack.md` — fonts dependency line
Swap the `@fontsource/inter` + `@fontsource/caveat` entries for `@fontsource` packages covering IBM
Plex Mono, IBM Plex Sans, Newsreader, Space Grotesk, Spline Sans Mono. Keep the "no Google Fonts CDN"
rationale (now reinforced, since the design's CDN usage is deliberately *not* adopted).

### 5. `docs/roadmap.md` — fold new UX into the phases
Add the `fontStyle` selector, the convert-event popover + calendar-link badge, and the
`showCalendarEvents` toggle into the relevant phases (settings/preferences + calendar-sync). Light
touch — these are spec additions, not a re-plan.

### 6. `CLAUDE.md` — quick-ref sync
Update the "Status" paragraph and any stack lines so the one-paragraph summary matches the new
design (fonts/selector, default accent, the new settings). Keep it a pointer to `/docs`.

### 7. `docs/README.md` — index note
Add a one-line note that the docs were re-derived from `/design` `Openweek v2 (paper).dc.html`
(the design source of truth), and refresh the Status blurb.

## Out of scope
- No application code (greenfield scaffold stays as-is; this is a docs/spec pass).
- The `Openweek v1 (sidebar).dc.html` / base `Openweek.dc.html` variants are **not** adopted — they
  are the superseded sidebar design and are ignored except as a "rejected v1 layout" footnote.

## Verification
- Re-read `docs/design.md` end-to-end: every token/hex matches the values extracted from
  `Openweek v2 (paper).dc.html`; no remaining Inter/Caveat/`#FAF7F0`/sidebar references
  (`grep -ri "inter\|caveat\|FAF7F0\|sidebar" docs/` should return only intentional "rejected v1"
  mentions).
- `data-model.md` tables list `fontStyle`/`accentColor`/`tagStyle`/`showCalendarEvents` and
  `tasks.startTime`/`linkedEventId`; header note no longer contradicts the tables.
- Cross-doc consistency: default accent reads **sky** everywhere; the font decision reads identically
  in `design.md`, `tech-stack.md`, `decisions.md`, and `CLAUDE.md`.
- Render check: open the rewritten `design.md` (Mermaid flow renders) and optionally screenshot for a
  visual skim.
