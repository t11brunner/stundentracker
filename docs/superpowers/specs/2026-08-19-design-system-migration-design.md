# Design system migration: ta-website → Stundentracker

## Goal

Re-skin `index.html` with the ETH TA Design System (sourced from the `ta-website` project, read-only reference — never modified). Single-file architecture, all JS logic, and all functional behavior stay exactly as-is. Only the `<style>` block, class names/markup needed to support the new visual patterns, and hardcoded color literals in `<script>` change.

## Non-goals

- No new features, no refactor of parsing/mapping/stats logic.
- No build step introduced (stays one static `index.html`).
- No dark mode — the source design system is light-only ("paper & ink"); Stundentracker adopts that as-is.
- `ta-website/` is a read-only reference. Nothing there is ever written to.

## Tokens (adopted verbatim from ta-website's production `index.html`, inline `:root`, no `@import`)

```css
:root{
  --paper:#eef1f2;
  --paper-2:#e3e8ea;
  --ink:#1d2326;
  --ink-soft:#565f63;
  --mute:#8b9497;
  --line:#d1d8da;
  --line-soft:#dde3e5;
  --accent: oklch(0.52 0.09 250);
  --accent-soft: oklch(0.52 0.09 250 / 0.1);
  --card:#f8fafb;
  --hatch: rgba(0,0,0,0.02);
  --f-head:'Public Sans', system-ui, sans-serif;
  --f-body:'Public Sans', system-ui, sans-serif;
  --f-mono:'IBM Plex Mono', monospace;
}
```

Fonts loaded via the same Google Fonts `<link>` pattern ta-website uses (`Public+Sans:wght@300;400;500;600;700` + `IBM+Plex+Mono:wght@400;500`), replacing the current DM Mono / DM Sans import.

Radii/spacing follow ta-website's literal values (not the abstracted token file): 14px cards/panels, 999px pills, 8px small controls, 6px inputs, 1px hairline borders — hardcoded per-rule like the source, for fidelity with the actual reference site's code style.

### Semantic/status colors (this app's own addition — the source system has none, since it has no equivalent "load level" concept)

Chosen as muted, harmonized extensions of the accent (same L/C family, rotated hue) rather than the current neon green/amber/red:

- Niedrig (low): `var(--accent)` — `oklch(0.52 0.09 250)`
- Normal: `oklch(0.58 0.08 170)` (muted teal/sage)
- Hoch (high): `oklch(0.62 0.10 70)` (muted amber/ochre)
- Sehr hoch (critical) / danger actions: `oklch(0.55 0.12 30)` (muted terracotta)

`btn-danger` and the unmapped-titles notice banner reuse the critical/high tones above instead of `#f06060`/`#f0c060`.

### Categorical chart palette (replaces `COLORS` neon array)

10 muted, harmonized hues at consistent lightness/chroma (same family as the accent, rotated around the wheel — "gedämpfte harmonierte Palette" per user decision):

```js
const COLORS = [
  'oklch(0.58 0.09 250)', // blue (accent family)
  'oklch(0.58 0.08 200)', // teal
  'oklch(0.60 0.07 150)', // sage
  'oklch(0.62 0.08 100)', // olive
  'oklch(0.62 0.09 60)',  // ochre
  'oklch(0.58 0.10 30)',  // terracotta
  'oklch(0.58 0.09 355)', // dusty rose
  'oklch(0.55 0.08 305)', // muted violet
  'oklch(0.50 0.08 230)', // deep blue-grey
  'oklch(0.65 0.06 170)', // pale teal-green
];
```

Used for: subject bars, doughnut chart segments, stacked month-chart segments.

Weekday-bars and hour-bars (previously `--accent2`/`--accent3`, now retired) get two fixed picks from this palette — `COLORS[0]` and `COLORS[5]` — chosen for max contrast against each other. Week-chart and day-chart (single-series time bars) use `var(--accent)` directly.

Chart.js text/grid colors switch from the dark-theme values (`#f0f0ee`, `#888884`, `rgba(255,255,255,0.04..06)`) to light-theme equivalents: tick/legend text → `var(--ink-soft)` (`#565f63`), gridlines → `var(--line-soft)` (`#dde3e5`).

## Typography shift

The source system reserves monospace **strictly** for numeric data readouts (percentages, hours, counts) — never for labels. Stundentracker currently uses mono for nearly everything. Reassignment:

- **Stays mono**: metric values (`.metric-value`), Belastung score (`#bel-score`), subject-bar hour readouts (`.bar-hours`), date-filter separator, subject/fach codes in the mapping table and bar rows (treated as short technical tags, same precedent as the source system's pole/zero root tags)
- **Moves to Public Sans, uppercase, 500 weight, +0.06em tracking, `--ink-soft`**: tab/nav labels, buttons, section labels (`.section-label`), table headers, "how-to" step labels, folder-status text, `.lbl`-style eyebrows
- **Body/headings**: Public Sans throughout, per source scale (h1 ~56px/700 masthead-style, section titles ~28px/600, body 15px/1.6–1.7 line-height)

## Layout

Header becomes a centered masthead (matching ta-website's `.masthead` pattern): eyebrow line ("Studienanalyse") + large Public Sans h1 ("Stundentracker"), no more left-aligned flex header. Tabs become a centered pill nav directly below (`nav.main` pattern: hairline top/bottom rule, pill items, active = accent color + `--accent-soft` background), replacing the current bottom-border tab strip.

## Component mapping

| Current | New treatment |
|---|---|
| `.tabs` / `.tab` | `nav.main` pill pattern |
| `.btn` / `.btn-accent` | pill button, hairline border, hover → accent border + `--accent-soft` bg (like `.dl-btn`) |
| `.btn-danger` | same pill shape, critical-tone border/text |
| `.drop-zone`, `.how-to`, folder panel | `--card` surface, hairline border, 14px radius |
| `.mapping-table` rows | hairline row borders (`.row`-style rhythm) instead of full-grid table borders |
| `.tag-pill` (fach code) | accent-tinted pill (accent-soft bg, accent text, hairline) |
| `.unknown-pill` | muted secondary pill, hover → accent |
| `.metrics` / `.metric` | `--card` tiles, hairline border, uppercase Public Sans label + mono value |
| `.panel` | `--card` surface, hairline border, 14px radius, no shadow |
| Belastung panel | mono score (colored by status), Public Sans status label, bar-track fill colored by status |
| `.date-input` | hairline border, `--card`/`--paper` bg, focus → accent border |

## Motion

140ms hover transitions (color/background/border). Tab switches animate as a 300ms opacity + `translateY(6px)` fade (`.screen.show`-style) instead of the current instant `display` toggle.

## Print styles

Simplify significantly: the base theme is already light/print-friendly, so the existing dark→light override block is no longer needed. Keep only: hide chrome (masthead nav, upload/mapping pages, top-bar, date-filter, unmapped-notice, pdf-btn) and ensure panels/charts render cleanly on white.

## Out of scope for this pass

- No changes to `ta-website/` under any circumstance.
- No changes to ICS parsing, RRULE expansion, mapping-rule storage format, IndexedDB folder handling, or Belastung scoring math.
