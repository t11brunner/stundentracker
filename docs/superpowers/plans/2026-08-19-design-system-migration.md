# Design System Migration Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Re-skin `index.html` (Stundentracker) with the ETH TA Design System sourced from `ta-website`, replacing the current dark neon theme with the paper/ink light theme, without changing any functional behavior.

**Architecture:** Single static `index.html` file, no build step, no test framework. All edits are CSS-rule and color-literal replacements within the existing `<style>` block and `<script>` block. "Tests" in this plan are `grep`-based structural assertions (a token either appears or doesn't, at an exact count) plus a manual browser visual check — the closest honest equivalent of TDD for a markup/CSS artifact with no unit-testable logic in play.

**Tech Stack:** Plain HTML/CSS/JS, Chart.js (CDN), Google Fonts (Public Sans, IBM Plex Mono), oklch() CSS colors.

## Global Constraints

- `ta-website/` (including `ta-website/.claude/skills/eth-ta-design/`) is read-only reference material. Never write to any path under it, at any point in any task.
- No changes to ICS parsing, RRULE expansion, mapping-rule storage format/localStorage key, IndexedDB folder handling, or the Belastung scoring formula (`calcBelastung`) — only the *colors* that logic outputs.
- No new files, no build step, no bundler, no framework. Everything stays inside `index.html`.
- No dark mode / no `prefers-color-scheme` branching — the design system is light-only.
- Fonts: `Public Sans` (weights 300;400;500;600;700) + `IBM Plex Mono` (weights 400;500), loaded from Google Fonts exactly as ta-website's production `index.html` loads them.
- Design tokens (exact values, from `docs/superpowers/specs/2026-08-19-design-system-migration-design.md`): `--paper:#eef1f2; --paper-2:#e3e8ea; --ink:#1d2326; --ink-soft:#565f63; --mute:#8b9497; --line:#d1d8da; --line-soft:#dde3e5; --accent: oklch(0.52 0.09 250); --accent-soft: oklch(0.52 0.09 250 / 0.1); --card:#f8fafb; --hatch: rgba(0,0,0,0.02);`
- Categorical chart palette (exact, ordered): `oklch(0.58 0.09 250)`, `oklch(0.58 0.08 200)`, `oklch(0.60 0.07 150)`, `oklch(0.62 0.08 100)`, `oklch(0.62 0.09 60)`, `oklch(0.58 0.10 30)`, `oklch(0.58 0.09 355)`, `oklch(0.55 0.08 305)`, `oklch(0.50 0.08 230)`, `oklch(0.65 0.06 170)`.
- Status colors (exact): low = `var(--accent)`; normal = `oklch(0.58 0.08 170)`; high = `oklch(0.62 0.10 70)`; critical = `oklch(0.55 0.12 30)`.
- Mono (`--f-mono` / IBM Plex Mono) is reserved for numeric data readouts only (hours, percentages, counts, subject/fach codes). Every label, tab, button, table header, and eyebrow uses Public Sans, uppercase, 500 weight, +0.06em tracking.
- Radii/spacing follow ta-website's literal per-rule values (14px cards/panels, 999px pills, 8px small controls, 1px hairlines) — hardcoded per rule, not abstracted into new spacing tokens.
- `git add index.html` only in every commit — never `git add -A` (the repo root has unrelated untracked `.ics`/`.json`/`.DS_Store` files that must stay out of these commits).
- Line numbers shift after every task (earlier tasks add/remove lines). Every task locates its target block with `grep -n` against the **current** file — never trust line numbers from an earlier task or from this plan's prose.

---

## Task 1: Design tokens, fonts, global variable rename, status tokens

**Files:**
- Modify: `index.html` — `<head>` font links, the `:root` block, the `body`/`::selection` rule, and every `var(--oldname)` occurrence file-wide.

**Interfaces:**
- Consumes: nothing (first task).
- Produces: the new token set available everywhere downstream — `--paper`, `--paper-2`, `--ink`, `--ink-soft`, `--mute`, `--line`, `--line-soft`, `--accent`, `--accent-soft`, `--card`, `--hatch`, `--f-head`, `--f-body`, `--f-mono`, `--status-normal`, `--status-high`, `--status-high-soft`, `--status-critical`, `--status-critical-soft`. Every later task assumes these exist and that no `var(--bg|--surface|--surface2|--border|--border2|--text|--muted|--mono|--sans)` reference remains anywhere in the file.

- [ ] **Step 1: Baseline check (must currently fail)**

Run:
```bash
cd /Users/t11brunner/Developer/stundentracker
grep -c "Public Sans" index.html; grep -c "var(--paper)" index.html; grep -c "var(--bg)" index.html
```
Expected: `0`, `0`, `1` — new tokens absent, old token present. This confirms the migration hasn't happened yet.

- [ ] **Step 2: Replace the font `<link>` tags**

Find (near top of `<head>`):
```html
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link href="https://fonts.googleapis.com/css2?family=DM+Mono:ital,wght@0,300;0,400;0,500;1,300&family=DM+Sans:wght@300;400;500&display=swap" rel="stylesheet">
```
Replace with:
```html
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Public+Sans:wght@300;400;500;600;700&family=IBM+Plex+Mono:wght@400;500&display=swap" rel="stylesheet">
```

- [ ] **Step 3: Replace the `:root` token block**

Find:
```css
    :root {
      --bg: #0e0e0f; --surface: #161618; --surface2: #1e1e21;
      --border: rgba(255,255,255,0.07); --border2: rgba(255,255,255,0.12);
      --text: #f0f0ee; --muted: #888884;
      --accent: #c8f060; --accent2: #60c8f0; --accent3: #f060a0;
      --mono: 'DM Mono', monospace; --sans: 'DM Sans', sans-serif;
    }
```
Replace with:
```css
    :root {
      /* ta-website ETH TA Design System — adopted verbatim */
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

      /* Stundentracker-specific additions — the source system has no "load level" concept */
      --status-normal: oklch(0.58 0.08 170);
      --status-high: oklch(0.62 0.10 70);
      --status-high-soft: oklch(0.62 0.10 70 / 0.1);
      --status-critical: oklch(0.55 0.12 30);
      --status-critical-soft: oklch(0.55 0.12 30 / 0.1);
    }
```

- [ ] **Step 4: Replace the `body` rule and add `::selection`**

Find:
```css
    body { background: var(--bg); color: var(--text); font-family: var(--sans); font-weight: 300; min-height: 100vh; }
```
Replace with:
```css
    body { background: var(--paper); color: var(--ink); font-family: var(--f-body); font-weight: 400; min-height: 100vh; -webkit-font-smoothing: antialiased; }
    ::selection { background: var(--accent-soft); }
```

- [ ] **Step 5: Global mechanical rename of every remaining variable usage**

This is a pure 1:1 text substitution across the whole file (CSS rules, inline `style="..."` attributes, and JS template-literal strings all use the identical `var(--name)` text). `--accent`, `--accent2`, `--accent3` are deliberately **excluded** — they're handled with semantic judgment in Tasks 2–8, not mechanically here.

Run:
```bash
cd /Users/t11brunner/Developer/stundentracker
sed -i '' \
  -e 's/var(--bg)/var(--paper)/g' \
  -e 's/var(--surface2)/var(--paper-2)/g' \
  -e 's/var(--surface)/var(--card)/g' \
  -e 's/var(--border2)/var(--line)/g' \
  -e 's/var(--border)/var(--line-soft)/g' \
  -e 's/var(--text)/var(--ink)/g' \
  -e 's/var(--muted)/var(--mute)/g' \
  -e 's/var(--mono)/var(--f-mono)/g' \
  -e 's/var(--sans)/var(--f-body)/g' \
  index.html
```

- [ ] **Step 6: Verify — exact expected counts**

Run:
```bash
cd /Users/t11brunner/Developer/stundentracker
for v in "var(--bg)" "var(--surface)" "var(--surface2)" "var(--border)" "var(--border2)" "var(--text)" "var(--muted)" "var(--mono)" "var(--sans)"; do
  echo "$v -> $(grep -o -F "$v" index.html | wc -l | tr -d ' ')"
done
for v in "var(--paper)" "var(--card)" "var(--paper-2)" "var(--line-soft)" "var(--line)" "var(--ink)" "var(--mute)" "var(--f-mono)" "var(--f-body)"; do
  echo "$v -> $(grep -o -F "$v" index.html | wc -l | tr -d ' ')"
done
```
Expected: every old-name line reads `-> 0`. New-name counts (excluding the one occurrence inside the `:root` block itself, since that line defines rather than references most of these — check actual numbers, they should all be ≥1): `var(--paper)` ≥1, `var(--card)` ≥7, `var(--paper-2)` ≥5, `var(--line-soft)` ≥18, `var(--line)` ≥4, `var(--ink)` ≥11, `var(--mute)` ≥26, `var(--f-mono)` ≥34, `var(--f-body)` ≥1.

Also run:
```bash
grep -c "DM Mono\|DM Sans" index.html   # expect 0 (font-family literal strings replaced too — verify none remain outside Chart.js, which Task 8 still owns)
grep -c "Public Sans" index.html        # expect >=1
```
If `DM Mono`/`DM Sans` literal strings still appear inside the Chart.js `<script>` block (e.g. `family:'DM Mono'`), that's expected — Task 8 owns those, leave them for now.

- [ ] **Step 7: Manual sanity — open in browser**

```bash
cd /Users/t11brunner/Developer/stundentracker
python3 -m http.server 8765 >/tmp/stundentracker-server.log 2>&1 &
```
Then use the browser preview tool to navigate to `http://localhost:8765/index.html` and screenshot. Expected: light paper background, Public Sans body text, no visibly broken/undefined-variable elements (undefined CSS vars render as if unset — watch for missing backgrounds/borders). The page will look visually unfinished (old CSS shapes/spacing still apply) — that's expected, later tasks polish it.

- [ ] **Step 8: Commit**

```bash
cd /Users/t11brunner/Developer/stundentracker
git add index.html
git commit -m "$(cat <<'EOF'
Adopt ta-website design tokens and fonts

Replaces the dark neon token set with the ETH TA Design System's
paper/ink tokens and Public Sans / IBM Plex Mono fonts. Mechanical
rename of every CSS variable usage file-wide; --accent/2/3 handled
separately in later tasks since they need semantic reassignment, not
a 1:1 rename.
EOF
)"
```

---

## Task 2: Header/masthead, pill nav, and page-switch motion

**Files:**
- Modify: `index.html` — header CSS + markup, `.tabs`/`.tab` CSS, `.page`/`.page.active` CSS.

**Interfaces:**
- Consumes: tokens from Task 1 (`--f-head`, `--mute`, `--ink`, `--accent`, `--accent-soft`, `--line-soft`).
- Produces: `.masthead` class (replaces `header`/inner flex `div`), restyled `.tabs`/`.tab` as a centered pill nav, `.page.active` now animates in.

- [ ] **Step 1: Locate current blocks**

```bash
cd /Users/t11brunner/Developer/stundentracker
grep -n "^    header \|^    \.logo \|^    h1 \|^    \.tabs \|^    \.tab \|^    \.tab:hover\|^    \.tab\.active\|^    \.page " index.html
grep -n "<header>\|<div class=\"tabs\">" index.html
```

- [ ] **Step 2: Replace header CSS**

Find:
```css
    header { padding: 3rem 2.5rem 2rem; border-bottom: 1px solid var(--line-soft); display: flex; align-items: flex-end; justify-content: space-between; gap: 2rem; flex-wrap: wrap; }
    .logo { font-family: var(--f-mono); font-size: 11px; color: var(--mute); letter-spacing: 0.12em; text-transform: uppercase; margin-bottom: 0.5rem; }
    h1 { font-family: var(--f-mono); font-size: clamp(1.8rem, 4vw, 2.8rem); font-weight: 300; letter-spacing: -0.02em; line-height: 1; }
    h1 span { color: var(--accent); }
```
Replace with:
```css
    .masthead { max-width: 640px; margin: 0 auto; padding: 4rem 1.5rem 0; text-align: center; }
    .logo { font-family: var(--f-head); font-weight: 500; font-size: 11px; color: var(--mute); letter-spacing: 0.06em; text-transform: uppercase; margin-bottom: 0.75rem; }
    h1 { font-family: var(--f-head); font-weight: 700; font-size: clamp(2rem, 5vw, 3rem); letter-spacing: -0.02em; line-height: 1.05; margin: 0; }
    h1 span { color: var(--accent); }
```

- [ ] **Step 3: Replace tabs CSS**

Find:
```css
    .tabs { display: flex; gap: 0; border-bottom: 1px solid var(--line-soft); padding: 0 2.5rem; }
    .tab { font-family: var(--f-mono); font-size: 11px; letter-spacing: 0.08em; color: var(--mute); padding: 1rem 1.25rem; cursor: pointer; border-bottom: 2px solid transparent; margin-bottom: -1px; transition: color 0.15s, border-color 0.15s; }
    .tab:hover { color: var(--ink); }
    .tab.active { color: var(--accent); border-bottom-color: var(--accent); }
```
Replace with:
```css
    .tabs { display: flex; justify-content: center; gap: 4px; margin: 2rem auto 2.5rem; flex-wrap: wrap; border-top: 1px solid var(--line-soft); border-bottom: 1px solid var(--line-soft); padding: 7px 0; max-width: 640px; }
    .tab { font-family: var(--f-head); font-weight: 500; font-size: 11px; letter-spacing: 0.06em; text-transform: uppercase; color: var(--mute); padding: 7px 14px; cursor: pointer; border-radius: 999px; transition: color .14s, background .14s; }
    .tab:hover { color: var(--ink); }
    .tab.active { color: var(--accent); background: var(--accent-soft); }
```

- [ ] **Step 4: Add page-switch fade animation**

Find:
```css
    .page { display: none; }
    .page.active { display: block; }
```
Replace with:
```css
    .page { display: none; }
    .page.active { display: block; animation: fade .3s ease; }
    @keyframes fade { from { opacity: 0; transform: translateY(6px); } to { opacity: 1; transform: none; } }
```
No JS change needed — `switchTab()` already just toggles the `.active` class, which now triggers this animation automatically.

- [ ] **Step 5: Update header markup**

Find:
```html
<header>
  <div>
    <div class="logo">Studienanalyse</div>
    <h1>Stunden<span>tracker</span></h1>
  </div>
</header>
```
Replace with:
```html
<div class="masthead">
  <div class="logo">Studienanalyse</div>
  <h1>Stunden<span>tracker</span></h1>
</div>
```
(Tab markup below it — the three `.tab` divs with their `onclick="switchTab(...)"` — stays byte-for-byte identical; only the CSS classes above changed shape, not the HTML structure or copy text.)

- [ ] **Step 6: Verify**

```bash
cd /Users/t11brunner/Developer/stundentracker
grep -c "<header>" index.html   # expect 0
grep -c "class=\"masthead\"" index.html   # expect 1
grep -c "^    header " index.html   # expect 0
```

- [ ] **Step 7: Visual check**

Reuse the server from Task 1 (`http://localhost:8765/index.html`, restart with `python3 -m http.server 8765 &` from the project root if it's no longer running). Screenshot: masthead should be centered with a large bold heading, pill nav below it with hairline top/bottom rules. Click each tab — content should fade+slide in.

- [ ] **Step 8: Commit**

```bash
cd /Users/t11brunner/Developer/stundentracker
git add index.html
git commit -m "$(cat <<'EOF'
Restyle header as centered masthead with pill nav

Replaces the left-aligned header/underline-tabs with ta-website's
centered masthead + nav.main pill pattern, and gives tab switches a
fade+slide transition instead of an instant display toggle.
EOF
)"
```

---

## Task 3: Buttons

**Files:**
- Modify: `index.html` — `.btn`, `.btn:hover`, `.btn-accent`, `.btn-sm`, `.btn-danger`, `.btn-danger:hover` CSS rules.

**Interfaces:**
- Consumes: tokens from Task 1, including `--status-critical` / `--status-critical-soft`.
- Produces: pill-shaped buttons matching the `.dl-btn` hover pattern; `.btn-danger` now uses the harmonized critical status color instead of neon red.

- [ ] **Step 1: Locate block**

```bash
cd /Users/t11brunner/Developer/stundentracker
grep -n "^    \.btn " index.html
```

- [ ] **Step 2: Replace**

Find:
```css
    .btn { display: inline-block; padding: 0.6rem 1.4rem; border: 1px solid var(--line); border-radius: 6px; font-family: var(--f-mono); font-size: 12px; color: var(--ink); background: transparent; cursor: pointer; letter-spacing: 0.06em; transition: border-color 0.2s, background 0.2s; }
    .btn:hover { border-color: var(--accent); background: rgba(200,240,96,0.07); }
    .btn-accent { border-color: var(--accent); color: var(--accent); }
    .btn-sm { padding: 0.35rem 0.8rem; font-size: 11px; }
    .btn-danger { border-color: rgba(240,96,96,0.4); color: #f06060; }
    .btn-danger:hover { border-color: #f06060; background: rgba(240,96,96,0.07); }
```
Replace with:
```css
    .btn { display: inline-block; padding: 8px 16px; border: 1px solid var(--line); border-radius: 999px; font-family: var(--f-head); font-weight: 500; font-size: 11px; letter-spacing: 0.06em; text-transform: uppercase; color: var(--ink); background: transparent; cursor: pointer; transition: border-color .14s, background .14s, color .14s; }
    .btn:hover { border-color: var(--accent); background: var(--accent-soft); }
    .btn-accent { border-color: var(--accent); color: var(--accent); }
    .btn-sm { padding: 6px 12px; font-size: 10px; }
    .btn-danger { color: var(--status-critical); }
    .btn-danger:hover { border-color: var(--status-critical); background: var(--status-critical-soft); }
```

- [ ] **Step 3: Verify**

```bash
cd /Users/t11brunner/Developer/stundentracker
grep -c "rgba(200,240,96\|rgba(240,96,96\|#f06060" index.html   # expect 0
grep -c "border-radius: 999px" index.html   # expect >=1 (this rule)
```

- [ ] **Step 4: Visual check**

On the running preview, check every button style: primary (`Dateien auswählen`, `Analysieren →`, `Speichern`), secondary (`Importieren`, `Exportieren`, `Neu laden`), and danger (`Trennen`, the `×` remove-file buttons, `entfernen` in the mapping table). All should render as pills; danger buttons should read in a muted terracotta, not bright red.

- [ ] **Step 5: Commit**

```bash
cd /Users/t11brunner/Developer/stundentracker
git add index.html
git commit -m "$(cat <<'EOF'
Restyle buttons as pills with harmonized danger color

Buttons match the ta-website dl-btn pill/hover pattern; btn-danger
now uses the muted --status-critical token instead of neon red.
EOF
)"
```

---

## Task 4: Upload page

**Files:**
- Modify: `index.html` — `#page-upload`, `.drop-zone`, `.drop-icon`, `.drop-title`, `.drop-sub`, `.how-to` (+ `h3`, `ol`, `li`, `li::before`, `li strong`) CSS; the `#file-list` inline-style `border-radius`.

**Interfaces:**
- Consumes: tokens from Task 1, pill/card conventions established in Tasks 2–3.
- Produces: upload page matching the card/hairline visual language (14px radii everywhere, labels moved to Public Sans).

- [ ] **Step 1: Locate block**

```bash
cd /Users/t11brunner/Developer/stundentracker
grep -n "^    #page-upload \|^    \.drop-zone\|^    \.drop-icon\|^    \.drop-title\|^    \.drop-sub\|^    \.how-to" index.html
grep -n 'id="file-list"' index.html
```

- [ ] **Step 2: Replace CSS block**

Find:
```css
    #page-upload { padding: 2.5rem; max-width: 640px; margin: 3rem auto; text-align: center; }
    .drop-zone { border: 1px dashed var(--line); border-radius: 12px; padding: 3rem 2rem; cursor: pointer; transition: border-color 0.2s, background 0.2s; }
    .drop-zone:hover, .drop-zone.drag-over { border-color: var(--accent); background: rgba(200,240,96,0.04); }
    .drop-icon { font-family: var(--f-mono); font-size: 2.5rem; color: var(--accent); margin-bottom: 1rem; display: block; }
    .drop-title { font-size: 1.1rem; font-weight: 400; margin-bottom: 0.5rem; }
    .drop-sub { font-size: 0.8rem; color: var(--mute); font-family: var(--f-mono); line-height: 1.6; }
    .how-to { margin-top: 2.5rem; text-align: left; background: var(--card); border: 1px solid var(--line-soft); border-radius: 10px; padding: 1.25rem 1.5rem; }
    .how-to h3 { font-family: var(--f-mono); font-size: 11px; letter-spacing: 0.1em; color: var(--mute); text-transform: uppercase; margin-bottom: 0.75rem; }
    .how-to ol { list-style: none; counter-reset: steps; }
    .how-to li { counter-increment: steps; font-size: 13px; color: var(--mute); padding: 0.3rem 0 0.3rem 1.75rem; position: relative; }
    .how-to li::before { content: counter(steps); position: absolute; left: 0; font-family: var(--f-mono); font-size: 11px; color: var(--accent); top: 0.35rem; }
    .how-to li strong { color: var(--ink); font-weight: 400; }
```
Replace with:
```css
    #page-upload { padding: 2.5rem 1.5rem; max-width: 640px; margin: 1rem auto 3rem; text-align: center; }
    .drop-zone { border: 1px dashed var(--line); border-radius: 14px; padding: 3rem 2rem; cursor: pointer; transition: border-color .14s, background .14s; }
    .drop-zone:hover, .drop-zone.drag-over { border-color: var(--accent); background: var(--accent-soft); }
    .drop-icon { font-family: var(--f-mono); font-size: 2rem; color: var(--accent); margin-bottom: 1rem; display: block; }
    .drop-title { font-family: var(--f-head); font-size: 1.1rem; font-weight: 600; margin-bottom: 0.5rem; }
    .drop-sub { font-size: 0.8rem; color: var(--mute); font-family: var(--f-body); line-height: 1.6; }
    .how-to { margin-top: 2.5rem; text-align: left; background: var(--card); border: 1px solid var(--line-soft); border-radius: 14px; padding: 1.25rem 1.5rem; }
    .how-to h3 { font-family: var(--f-head); font-weight: 500; font-size: 11px; letter-spacing: 0.06em; color: var(--mute); text-transform: uppercase; margin-bottom: 0.75rem; }
    .how-to ol { list-style: none; counter-reset: steps; }
    .how-to li { counter-increment: steps; font-size: 13px; color: var(--ink-soft); padding: 0.3rem 0 0.3rem 1.75rem; position: relative; }
    .how-to li::before { content: counter(steps); position: absolute; left: 0; font-family: var(--f-mono); font-size: 11px; color: var(--accent); top: 0.35rem; }
    .how-to li strong { color: var(--ink); font-weight: 500; }
```

- [ ] **Step 3: Bump file-list card radius**

Find (inline style, inside the `#file-list` div):
```
border-radius:10px;
```
(the one on the `<div id="file-list" ...>` line — verify with the grep from Step 1 that you're editing the right occurrence, there are other `border-radius:10px` inline styles elsewhere in the file that must NOT be touched here). Replace with:
```
border-radius:14px;
```

- [ ] **Step 4: Verify**

```bash
cd /Users/t11brunner/Developer/stundentracker
grep -c "rgba(200,240,96,0.04)" index.html   # expect 0
```

- [ ] **Step 5: Visual check**

On the preview: upload page drop-zone should have a 14px-rounded dashed border that highlights blue on hover; the "how-to" panel and folder-connect panel should look like flat cards with hairline borders, step numbers in accent blue, labels in uppercase Public Sans (not mono).

- [ ] **Step 6: Commit**

```bash
cd /Users/t11brunner/Developer/stundentracker
git add index.html
git commit -m "Restyle upload page to card/hairline visual language"
```

---

## Task 5: Mapping page

**Files:**
- Modify: `index.html` — `#page-mapping`, `.mapping-header h2`, `.mapping-desc` (+`code`), `.add-row` (+`label`,`input`,`input:focus`), `.mapping-table` (+`th`,`td`,`tbody tr:hover td`), `.tag-pill`, `.pattern-text`, `.unknown-section` (+`title`,`list`), `.unknown-pill` (+`:hover`) CSS.

**Interfaces:**
- Consumes: tokens from Task 1, `--accent`/`--accent-soft` (replacing the retired `--accent2`).
- Produces: mapping page matching the card/pill visual language; `mapping-desc code` and `.add-row input:focus` and `.unknown-pill:hover` no longer reference the removed `--accent2`.

- [ ] **Step 1: Locate block**

```bash
cd /Users/t11brunner/Developer/stundentracker
grep -n "^    #page-mapping \|^    \.mapping-\|^    \.add-row\|^    \.tag-pill\|^    \.pattern-text\|^    \.unknown-" index.html
```

- [ ] **Step 2: Replace**

Find:
```css
    #page-mapping { padding: 2.5rem; }
    .mapping-header { display: flex; align-items: center; justify-content: space-between; margin-bottom: 1.5rem; flex-wrap: wrap; gap: 1rem; }
    .mapping-header h2 { font-family: var(--f-mono); font-size: 14px; font-weight: 400; color: var(--ink); }
    .mapping-desc { font-size: 13px; color: var(--mute); margin-bottom: 1.5rem; line-height: 1.8; background: var(--card); border: 1px solid var(--line-soft); border-radius: 8px; padding: 1rem 1.25rem; }
    .mapping-desc code { font-family: var(--f-mono); color: var(--accent2); font-size: 12px; background: rgba(96,200,240,0.08); padding: 1px 5px; border-radius: 3px; }
    .add-row { display: flex; gap: 0.75rem; align-items: center; flex-wrap: wrap; margin-bottom: 1.25rem; padding: 1rem 1.25rem; background: var(--card); border: 1px solid var(--line-soft); border-radius: 8px; }
    .add-row label { font-family: var(--f-mono); font-size: 10px; color: var(--mute); letter-spacing: 0.08em; text-transform: uppercase; white-space: nowrap; }
    .add-row input { background: var(--paper-2); border: 1px solid var(--line); border-radius: 6px; font-family: var(--f-mono); font-size: 12px; color: var(--ink); padding: 0.45rem 0.75rem; outline: none; transition: border-color 0.2s; }
    .add-row input:focus { border-color: var(--accent2); }
    .add-row input::placeholder { color: var(--mute); }
    .add-row input.w-pattern { width: 240px; }
    .add-row input.w-subj { width: 100px; }

    .mapping-table { width: 100%; border-collapse: collapse; margin-bottom: 1.5rem; }
    .mapping-table th { font-family: var(--f-mono); font-size: 10px; letter-spacing: 0.1em; color: var(--mute); text-transform: uppercase; text-align: left; padding: 0.5rem 0.75rem; border-bottom: 1px solid var(--line-soft); }
    .mapping-table td { padding: 0.5rem 0.75rem; border-bottom: 1px solid var(--line-soft); vertical-align: middle; font-family: var(--f-mono); font-size: 12px; }
    .mapping-table tbody tr:hover td { background: var(--paper-2); }
    .tag-pill { display: inline-block; font-family: var(--f-mono); font-size: 11px; padding: 2px 8px; border-radius: 20px; background: rgba(200,240,96,0.1); color: var(--accent); border: 1px solid rgba(200,240,96,0.2); }
    .pattern-text { color: var(--ink); }

    .unknown-section { margin-top: 2rem; padding-top: 1.5rem; border-top: 1px solid var(--line-soft); }
    .unknown-title { font-family: var(--f-mono); font-size: 10px; letter-spacing: 0.1em; color: var(--mute); text-transform: uppercase; margin-bottom: 0.75rem; }
    .unknown-list { display: flex; flex-wrap: wrap; gap: 6px; }
    .unknown-pill { font-family: var(--f-mono); font-size: 11px; padding: 3px 10px; border-radius: 20px; background: var(--paper-2); color: var(--mute); cursor: pointer; border: 1px solid var(--line-soft); transition: border-color 0.15s, color 0.15s; }
    .unknown-pill:hover { border-color: var(--accent2); color: var(--accent2); }
```
Replace with:
```css
    #page-mapping { padding: 2.5rem 1.5rem; max-width: 760px; margin: 0 auto; }
    .mapping-header { display: flex; align-items: center; justify-content: space-between; margin-bottom: 1.5rem; flex-wrap: wrap; gap: 1rem; }
    .mapping-header h2 { font-family: var(--f-head); font-size: 20px; font-weight: 600; color: var(--ink); letter-spacing: -0.01em; }
    .mapping-desc { font-size: 13px; color: var(--ink-soft); margin-bottom: 1.5rem; line-height: 1.8; background: var(--card); border: 1px solid var(--line-soft); border-radius: 10px; padding: 1rem 1.25rem; }
    .mapping-desc code { font-family: var(--f-mono); color: var(--accent); font-size: 12px; background: var(--accent-soft); padding: 1px 5px; border-radius: 4px; }
    .add-row { display: flex; gap: 0.75rem; align-items: center; flex-wrap: wrap; margin-bottom: 1.25rem; padding: 1rem 1.25rem; background: var(--card); border: 1px solid var(--line-soft); border-radius: 10px; }
    .add-row label { font-family: var(--f-head); font-weight: 500; font-size: 10px; color: var(--mute); letter-spacing: 0.06em; text-transform: uppercase; white-space: nowrap; }
    .add-row input { background: var(--paper); border: 1px solid var(--line); border-radius: 8px; font-family: var(--f-mono); font-size: 12px; color: var(--ink); padding: 0.45rem 0.75rem; outline: none; transition: border-color .14s; }
    .add-row input:focus { border-color: var(--accent); }
    .add-row input::placeholder { color: var(--mute); }
    .add-row input.w-pattern { width: 240px; }
    .add-row input.w-subj { width: 100px; }

    .mapping-table { width: 100%; border-collapse: collapse; margin-bottom: 1.5rem; }
    .mapping-table th { font-family: var(--f-head); font-weight: 500; font-size: 10px; letter-spacing: 0.06em; color: var(--mute); text-transform: uppercase; text-align: left; padding: 0.5rem 0.75rem; border-bottom: 1px solid var(--line-soft); }
    .mapping-table td { padding: 0.5rem 0.75rem; border-bottom: 1px solid var(--line-soft); vertical-align: middle; font-family: var(--f-body); font-size: 13px; }
    .mapping-table tbody tr:hover td { background: var(--card); }
    .tag-pill { display: inline-block; font-family: var(--f-mono); font-size: 11px; padding: 2px 8px; border-radius: 999px; background: var(--accent-soft); color: var(--accent); border: 1px solid transparent; }
    .pattern-text { color: var(--ink); }

    .unknown-section { margin-top: 2rem; padding-top: 1.5rem; border-top: 1px solid var(--line-soft); }
    .unknown-title { font-family: var(--f-head); font-weight: 500; font-size: 10px; letter-spacing: 0.06em; color: var(--mute); text-transform: uppercase; margin-bottom: 0.75rem; }
    .unknown-list { display: flex; flex-wrap: wrap; gap: 6px; }
    .unknown-pill { font-family: var(--f-mono); font-size: 11px; padding: 3px 10px; border-radius: 999px; background: var(--paper-2); color: var(--mute); cursor: pointer; border: 1px solid var(--line-soft); transition: border-color .14s, color .14s; }
    .unknown-pill:hover { border-color: var(--accent); color: var(--accent); }
```

- [ ] **Step 3: Verify**

```bash
cd /Users/t11brunner/Developer/stundentracker
grep -c "accent2" index.html   # expect a lower count than before this task (mapping-page's 4 uses gone); nonzero remainder is expected (Tasks 6/7 haven't run yet)
grep -c "rgba(200,240,96\|rgba(96,200,240" index.html   # expect 0
```

- [ ] **Step 4: Visual check**

On the preview: mapping tab — add-row inputs should have a light border that turns blue on focus, table rows should show fach codes as blue accent pills, unknown-title pills should turn blue on hover.

- [ ] **Step 5: Commit**

```bash
cd /Users/t11brunner/Developer/stundentracker
git add index.html
git commit -m "Restyle mapping page and retire --accent2 usages"
```

---

## Task 6: Stats page shell (top-bar, metrics, panels, date filter, unmapped notice)

**Files:**
- Modify: `index.html` — `#page-stats`, `.top-bar`, `.filename`, `.metrics`, `.metric*`, `.section-label`, `.grid-2`, `.panel*`, `.subject-bar`/`.subject-code`/`.bar-track`/`.bar-fill`/`.bar-hours`, `.chart-wrap`, `.unmapped-notice`, `#date-filter`, `.date-input*` CSS.

**Interfaces:**
- Consumes: tokens from Task 1, including `--status-high` / `--status-high-soft`.
- Produces: stats shell matching the card/hairline language; `.unmapped-notice` uses the harmonized high-status color instead of neon amber; `.date-input` correctly declares `color-scheme: light`.

- [ ] **Step 1: Locate block**

```bash
cd /Users/t11brunner/Developer/stundentracker
grep -n "^    #page-stats \|^    \.top-bar\|^    \.filename\|^    \.metrics\|^    \.metric \|^    \.metric-\|^    \.section-label\|^    \.grid-2\|^    \.panel\|^    \.subject-bar\|^    \.subject-code\|^    \.bar-track\|^    \.bar-fill\|^    \.bar-hours\|^    \.chart-wrap\|^    \.unmapped-notice\|^    #date-filter\|^    \.date-input" index.html
```

- [ ] **Step 2: Replace**

Find:
```css
    #page-stats { padding: 0 2.5rem 4rem; }
    .top-bar { display: flex; align-items: center; justify-content: space-between; padding: 1.5rem 0; border-bottom: 1px solid var(--line-soft); margin-bottom: 2rem; flex-wrap: wrap; gap: 1rem; }
    .filename { font-family: var(--f-mono); font-size: 12px; color: var(--mute); }
    .filename span { color: var(--accent); }
    .metrics { display: grid; grid-template-columns: repeat(auto-fit, minmax(140px, 1fr)); gap: 1px; background: var(--line-soft); border: 1px solid var(--line-soft); border-radius: 10px; overflow: hidden; margin-bottom: 2.5rem; }
    .metric { background: var(--card); padding: 1.25rem 1.5rem; }
    .metric-label { font-family: var(--f-mono); font-size: 10px; color: var(--mute); letter-spacing: 0.1em; text-transform: uppercase; margin-bottom: 0.5rem; }
    .metric-value { font-family: var(--f-mono); font-size: 1.8rem; font-weight: 400; color: var(--accent); line-height: 1; }
    .metric-sub { font-size: 11px; color: var(--mute); margin-top: 0.25rem; }
    .section-label { font-family: var(--f-mono); font-size: 10px; letter-spacing: 0.12em; color: var(--mute); text-transform: uppercase; margin-bottom: 1rem; }
    .grid-2 { display: grid; grid-template-columns: 1fr 1fr; gap: 1.5rem; margin-bottom: 2rem; }
    @media (max-width: 700px) { .grid-2 { grid-template-columns: 1fr; } }
    .panel { background: var(--card); border: 1px solid var(--line-soft); border-radius: 10px; padding: 1.5rem; }
    .panel.full { margin-bottom: 2rem; }
    .subject-bar { display: flex; align-items: center; gap: 10px; margin-bottom: 10px; font-family: var(--f-mono); font-size: 12px; }
    .subject-code { width: 80px; color: var(--ink); flex-shrink: 0; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
    .bar-track { flex: 1; height: 6px; background: var(--paper-2); border-radius: 3px; overflow: hidden; }
    .bar-fill { height: 100%; border-radius: 3px; transition: width 0.6s cubic-bezier(.22,1,.36,1); }
    .bar-hours { width: 48px; text-align: right; color: var(--mute); }
    .chart-wrap { position: relative; width: 100%; height: 220px; }
    .unmapped-notice { background: rgba(240,192,96,0.08); border: 1px solid rgba(240,192,96,0.2); border-radius: 8px; padding: 0.75rem 1rem; margin-bottom: 1.5rem; font-family: var(--f-mono); font-size: 12px; color: #f0c060; display: flex; align-items: center; justify-content: space-between; gap: 1rem; flex-wrap: wrap; }

    /* DATE FILTER */
    #date-filter { display: flex; align-items: center; gap: 0.75rem; flex-wrap: wrap; padding: 0.75rem 1rem; background: var(--card); border: 1px solid var(--line-soft); border-radius: 8px; margin-bottom: 1.5rem; }
    .date-input { background: var(--paper-2); border: 1px solid var(--line); border-radius: 6px; font-family: var(--f-mono); font-size: 12px; color: var(--ink); padding: 0.4rem 0.6rem; outline: none; transition: border-color 0.2s; color-scheme: dark; }
    .date-input:focus { border-color: var(--accent2); }
```
Replace with:
```css
    #page-stats { padding: 0 1.5rem 4rem; max-width: 980px; margin: 0 auto; }
    .top-bar { display: flex; align-items: center; justify-content: space-between; padding: 1.5rem 0; border-bottom: 1px solid var(--line-soft); margin-bottom: 2rem; flex-wrap: wrap; gap: 1rem; }
    .filename { font-family: var(--f-head); font-weight: 500; font-size: 11px; letter-spacing: 0.04em; color: var(--mute); }
    .filename span { color: var(--accent); }
    .metrics { display: grid; grid-template-columns: repeat(auto-fit, minmax(140px, 1fr)); gap: 1px; background: var(--line-soft); border: 1px solid var(--line-soft); border-radius: 14px; overflow: hidden; margin-bottom: 2.5rem; }
    .metric { background: var(--card); padding: 1.25rem 1.5rem; }
    .metric-label { font-family: var(--f-head); font-weight: 500; font-size: 10px; color: var(--mute); letter-spacing: 0.06em; text-transform: uppercase; margin-bottom: 0.5rem; }
    .metric-value { font-family: var(--f-mono); font-size: 1.8rem; font-weight: 500; color: var(--accent); line-height: 1; }
    .metric-sub { font-size: 11px; color: var(--mute); margin-top: 0.25rem; }
    .section-label { font-family: var(--f-head); font-weight: 500; font-size: 10px; letter-spacing: 0.06em; color: var(--mute); text-transform: uppercase; margin-bottom: 1rem; }
    .grid-2 { display: grid; grid-template-columns: 1fr 1fr; gap: 1.5rem; margin-bottom: 2rem; }
    @media (max-width: 700px) { .grid-2 { grid-template-columns: 1fr; } }
    .panel { background: var(--card); border: 1px solid var(--line-soft); border-radius: 14px; padding: 1.5rem; }
    .panel.full { margin-bottom: 2rem; }
    .subject-bar { display: flex; align-items: center; gap: 10px; margin-bottom: 10px; font-family: var(--f-mono); font-size: 12px; }
    .subject-code { width: 80px; color: var(--ink); flex-shrink: 0; overflow: hidden; text-overflow: ellipsis; white-space: nowrap; }
    .bar-track { flex: 1; height: 6px; background: var(--paper-2); border-radius: 3px; overflow: hidden; }
    .bar-fill { height: 100%; border-radius: 3px; transition: width 0.6s cubic-bezier(.22,1,.36,1); }
    .bar-hours { width: 48px; text-align: right; color: var(--mute); }
    .chart-wrap { position: relative; width: 100%; height: 220px; }
    .unmapped-notice { background: var(--status-high-soft); border: 1px solid var(--status-high); border-radius: 10px; padding: 0.75rem 1rem; margin-bottom: 1.5rem; font-family: var(--f-body); font-weight: 500; font-size: 13px; color: var(--status-high); display: flex; align-items: center; justify-content: space-between; gap: 1rem; flex-wrap: wrap; }

    /* DATE FILTER */
    #date-filter { display: flex; align-items: center; gap: 0.75rem; flex-wrap: wrap; padding: 0.75rem 1rem; background: var(--card); border: 1px solid var(--line-soft); border-radius: 10px; margin-bottom: 1.5rem; }
    .date-input { background: var(--paper); border: 1px solid var(--line); border-radius: 8px; font-family: var(--f-mono); font-size: 12px; color: var(--ink); padding: 0.4rem 0.6rem; outline: none; transition: border-color .14s; color-scheme: light; }
    .date-input:focus { border-color: var(--accent); }
```

- [ ] **Step 3: Verify**

```bash
cd /Users/t11brunner/Developer/stundentracker
grep -c "rgba(240,192,96\|#f0c060\|color-scheme: dark" index.html   # expect 0 for these three (the JS Belastung #f0c060 literal is handled in Task 7 — if this grep still shows a hit, confirm it's only line ~944/945 in the <script>, not here)
```

- [ ] **Step 4: Visual check**

Load one of the repo's sample calendars (`ETH Präsenz.ics` or `ETH Selbststudium.ics`) via the drop zone, go to Statistiken. Metric tiles, panels, date filter, and (if any unmapped titles exist) the unmapped-notice banner should all read as light cards with hairline borders; the notice banner should be a muted amber/ochre tone, not neon.

- [ ] **Step 5: Commit**

```bash
cd /Users/t11brunner/Developer/stundentracker
git add index.html
git commit -m "Restyle stats page shell (metrics, panels, date filter, notice)"
```

---

## Task 7: Belastung status colors and weekday/hour bar colors (JS)

**Files:**
- Modify: `index.html` — inside `<script>`: the `weekday-bars` and `hour-bars` template literals (`renderStats`), and `renderBelastung`'s color assignments.

**Interfaces:**
- Consumes: `--status-normal`, `--status-high`, `--status-critical` from Task 1; palette entries `oklch(0.58 0.09 250)` (index 0) and `oklch(0.58 0.10 30)` (index 5) from the Global Constraints palette.
- Produces: Belastung panel and weekday/hour bar charts with no remaining reference to the retired `--accent2`/`--accent3`, and no neon `#f0c060`/`#f06060` literals.

- [ ] **Step 1: Locate block**

```bash
cd /Users/t11brunner/Developer/stundentracker
grep -n "background:var(--accent2)\|background:var(--accent3)\|function renderBelastung" index.html
```

- [ ] **Step 2: Fix weekday-bars fill color**

Find:
```
    <div class="bar-track"><div class="bar-fill" style="width:${Math.round(wdH[i]/wdMax*100)}%;background:var(--accent2)"></div></div>
```
Replace with:
```
    <div class="bar-track"><div class="bar-fill" style="width:${Math.round(wdH[i]/wdMax*100)}%;background:oklch(0.58 0.09 250)"></div></div>
```
(This is inside an inline `style="..."` attribute rendered into the page's HTML, so it must be a value the browser's CSS engine can resolve directly — a literal color, not a variable that no longer exists.)

- [ ] **Step 3: Fix hour-bars fill color**

Find:
```
    <div class="bar-track"><div class="bar-fill" style="width:${Math.round(h/hMax*100)}%;background:var(--accent3)"></div></div>
```
Replace with:
```
    <div class="bar-track"><div class="bar-fill" style="width:${Math.round(h/hMax*100)}%;background:oklch(0.58 0.10 30)"></div></div>
```

- [ ] **Step 4: Fix Belastung status colors**

Find:
```js
  if (score < 40)        { label = 'Niedrige Belastung';    color = 'var(--accent)';  text = 'Aktuell ruhig – Reserve für intensivere Wochen.'; }
  else if (score < 70)   { label = 'Normale Belastung';     color = 'var(--accent2)'; text = 'Dein üblicher Rhythmus.'; }
  else if (score < 90)   { label = 'Hohe Belastung';        color = '#f0c060';        text = 'Plane bewusst Pausen, sonst kippt\'s.'; }
  else                   { label = 'Sehr hohe Belastung';   color = '#f06060';        text = 'Gönn dir Erholung – langfristig nicht durchhaltbar.'; }
```
Replace with:
```js
  if (score < 40)        { label = 'Niedrige Belastung';    color = 'var(--accent)';          text = 'Aktuell ruhig – Reserve für intensivere Wochen.'; }
  else if (score < 70)   { label = 'Normale Belastung';     color = 'var(--status-normal)';   text = 'Dein üblicher Rhythmus.'; }
  else if (score < 90)   { label = 'Hohe Belastung';        color = 'var(--status-high)';     text = 'Plane bewusst Pausen, sonst kippt\'s.'; }
  else                   { label = 'Sehr hohe Belastung';   color = 'var(--status-critical)'; text = 'Gönn dir Erholung – langfristig nicht durchhaltbar.'; }
```
(These four ARE valid as CSS variable references — unlike the bar fills above, they're applied via `scoreEl.style.color = color` / `barEl.style.background = color`, i.e. through the DOM/CSS engine, which resolves `var()` correctly. Only literal-canvas contexts, per Task 8, can't use `var()`.)

- [ ] **Step 5: Verify**

```bash
cd /Users/t11brunner/Developer/stundentracker
grep -c "accent2\|accent3" index.html   # expect 0 — every usage across the whole file is now retired
grep -c "#f0c060\|#f06060" index.html   # expect 0
```

- [ ] **Step 6: Functional + visual check**

Load a sample `.ics` from the repo root, go to Statistiken. Weekday-bars and hour-bars panels should render with two distinguishable colors from the harmonized palette (not the old cyan/pink). Belastung panel: the score/label color should shift between accent-blue, teal, ochre, and terracotta depending on the computed score — you can sanity-check the branch logic by temporarily reading `calcBelastung`'s output in the console (`renderBelastung` is called from `renderStats`; no need to force each branch, just confirm the panel renders with one of the four colors and no console errors).

- [ ] **Step 7: Commit**

```bash
cd /Users/t11brunner/Developer/stundentracker
git add index.html
git commit -m "Retire accent2/accent3 in Belastung and weekday/hour bar JS"
```

---

## Task 8: Chart.js literal colors (COLORS palette, legend/tick/grid, week/day charts)

**Files:**
- Modify: `index.html` — inside `<script>`: the `COLORS` array, the pie-chart `legend.labels` config, the `month-chart`/`week-chart`/`day-chart` `scales`/`datasets` configs.

**Interfaces:**
- Consumes: the exact 10-entry categorical palette and accent literal from Global Constraints.
- Produces: every Chart.js color/font config using the new palette and Public-Sans-for-labels/mono-for-numbers split; zero remaining `DM Mono` or neon hex literals anywhere in the file.

- [ ] **Step 1: Locate blocks**

```bash
cd /Users/t11brunner/Developer/stundentracker
grep -n "const COLORS\|DM Mono\|#f0f0ee\|#888884\|rgba(255,255,255" index.html
```

- [ ] **Step 2: Replace the COLORS array**

Find:
```js
const COLORS = ['#c8f060','#60c8f0','#f060a0','#f0c060','#60f0a0','#a060f0','#f09060','#60f0f0','#f060f0','#90f060'];
```
Replace with:
```js
const COLORS = [
  'oklch(0.58 0.09 250)', 'oklch(0.58 0.08 200)', 'oklch(0.60 0.07 150)', 'oklch(0.62 0.08 100)', 'oklch(0.62 0.09 60)',
  'oklch(0.58 0.10 30)',  'oklch(0.58 0.09 355)', 'oklch(0.55 0.08 305)', 'oklch(0.50 0.08 230)', 'oklch(0.65 0.06 170)',
];
```

- [ ] **Step 3: Fix pie-chart legend labels**

Find:
```js
        labels:{ color:'#f0f0ee', font:{family:'DM Mono, monospace',size:11}, boxWidth:12, boxHeight:12, padding:16, usePointStyle:true, pointStyle:'rectRounded' }
```
Replace with:
```js
        labels:{ color:'#565f63', font:{family:'Public Sans, sans-serif',size:11,weight:'500'}, boxWidth:12, boxHeight:12, padding:16, usePointStyle:true, pointStyle:'rectRounded' }
```
(These labels name each subject — text, not a number — so they move to Public Sans, matching the label-vs-readout split from the design system.)

- [ ] **Step 4: Fix month-chart scales**

Find:
```js
      scales:{ x:{stacked:true, ticks:{color:'#888884',font:{family:'DM Mono',size:10},autoSkip:false,maxRotation:45},grid:{color:'rgba(255,255,255,0.04)'}}, y:{stacked:true, ticks:{color:'#888884',font:{family:'DM Mono',size:10},callback:v=>v+'h'},grid:{color:'rgba(255,255,255,0.06)'}}}}
```
Replace with:
```js
      scales:{ x:{stacked:true, ticks:{color:'#8b9497',font:{family:'Public Sans',size:10},autoSkip:false,maxRotation:45},grid:{color:'#dde3e5'}}, y:{stacked:true, ticks:{color:'#8b9497',font:{family:'IBM Plex Mono',size:10},callback:v=>v+'h'},grid:{color:'#d1d8da'}}}}
```
(x-axis ticks are month names — text, Public Sans. y-axis ticks are hour counts — numeric, IBM Plex Mono.)

- [ ] **Step 5: Fix week-chart**

Find:
```js
    data: { labels: wkLabels, datasets: [{ data: wks.map(k=>+weekMap[k].toFixed(2)), backgroundColor: '#c8f060', borderWidth:0, borderRadius:2 }] },
```
Replace with:
```js
    data: { labels: wkLabels, datasets: [{ data: wks.map(k=>+weekMap[k].toFixed(2)), backgroundColor: 'oklch(0.52 0.09 250)', borderWidth:0, borderRadius:2 }] },
```

Find:
```js
        x:{ ticks:{color:'#888884',font:{family:'DM Mono',size:10},maxRotation:45,autoSkip:true,maxTicksLimit:26}, grid:{color:'rgba(255,255,255,0.04)'} },
        y:{ ticks:{color:'#888884',font:{family:'DM Mono',size:10},callback:v=>v+'h'}, grid:{color:'rgba(255,255,255,0.06)'}, beginAtZero:true }
```
(this exact pair appears twice in the file — once for week-chart, once for day-chart; edit the one inside the week-chart block located in Step 1's grep, identifiable by `maxTicksLimit:26` on the x-axis line)
Replace with:
```js
        x:{ ticks:{color:'#8b9497',font:{family:'Public Sans',size:10},maxRotation:45,autoSkip:true,maxTicksLimit:26}, grid:{color:'#dde3e5'} },
        y:{ ticks:{color:'#8b9497',font:{family:'IBM Plex Mono',size:10},callback:v=>v+'h'}, grid:{color:'#d1d8da'}, beginAtZero:true }
```

- [ ] **Step 6: Fix day-chart**

Find:
```js
    data: { labels: dayLabels, datasets: [{ data: days.map(d=>+dayMap[d].toFixed(2)), backgroundColor: '#60c8f0', borderWidth:0, borderRadius:2 }] },
```
Replace with:
```js
    data: { labels: dayLabels, datasets: [{ data: days.map(d=>+dayMap[d].toFixed(2)), backgroundColor: 'oklch(0.52 0.09 250)', borderWidth:0, borderRadius:2 }] },
```

Find the remaining `maxTicksLimit:40` pair:
```js
        x:{ ticks:{color:'#888884',font:{family:'DM Mono',size:10},maxRotation:45,autoSkip:true,maxTicksLimit:40}, grid:{color:'rgba(255,255,255,0.04)'} },
        y:{ ticks:{color:'#888884',font:{family:'DM Mono',size:10},callback:v=>v+'h'}, grid:{color:'rgba(255,255,255,0.06)'}, beginAtZero:true }
```
Replace with:
```js
        x:{ ticks:{color:'#8b9497',font:{family:'Public Sans',size:10},maxRotation:45,autoSkip:true,maxTicksLimit:40}, grid:{color:'#dde3e5'} },
        y:{ ticks:{color:'#8b9497',font:{family:'IBM Plex Mono',size:10},callback:v=>v+'h'}, grid:{color:'#d1d8da'}, beginAtZero:true }
```

- [ ] **Step 7: Verify — full-file regression sweep**

```bash
cd /Users/t11brunner/Developer/stundentracker
grep -c "DM Mono\|DM Sans\|#c8f060\|#60c8f0\|#f060a0\|#f0c060\|#60f0a0\|#a060f0\|#f09060\|#60f0f0\|#f060f0\|#90f060\|#f0f0ee\|#888884\|rgba(255,255,255\|accent2\|accent3\|#f06060" index.html
```
Expected: `0`. This is the final, whole-file confirmation that every dark-theme color literal is gone.

- [ ] **Step 8: Visual check**

Load a sample `.ics`. Every chart (doughnut, stacked month bars, week bars, day bars) should render in the new muted palette; legend text and axis labels should be readable dark-grey on the paper background, not invisible/mismatched.

- [ ] **Step 9: Commit**

```bash
cd /Users/t11brunner/Developer/stundentracker
git add index.html
git commit -m "$(cat <<'EOF'
Replace Chart.js color literals with harmonized light-theme palette

Covers the categorical COLORS array, legend/tick/grid colors, and the
week/day chart accent color. Chart.js configs use literal colors
(not CSS var()) since canvas rendering can't resolve custom
properties — this mirrors how the original dark theme also hardcoded
its chart literals rather than using var().
EOF
)"
```

---

## Task 9: Print styles

**Files:**
- Modify: `index.html` — the `@media print { ... }` block.

**Interfaces:**
- Consumes: `.masthead` class name from Task 2 (the print block's chrome-hiding selector list must reference the renamed element).
- Produces: a much shorter print block, since the base theme is already print-appropriate.

- [ ] **Step 1: Locate block**

```bash
cd /Users/t11brunner/Developer/stundentracker
grep -n "@media print" index.html
```

- [ ] **Step 2: Replace**

Find:
```css
    @media print {
      header, .tabs, #page-upload, #page-mapping,
      .top-bar, #date-filter, #unmapped-notice,
      #pdf-btn { display: none !important; }
      #page-stats { display: block !important; padding: 0; }
      body { background: #fff; color: #111; }
      .panel { background: #f8f8f8 !important; border: 1px solid #ddd !important; break-inside: avoid; }
      .metric { background: #f8f8f8 !important; }
      .metrics { background: #ddd !important; }
      .bar-track { background: #e0e0e0 !important; }
      .section-label, .metric-label { color: #666 !important; }
      .metric-value { color: #333 !important; }
      .bar-hours, .subject-code { color: #555 !important; }
      .grid-2 { grid-template-columns: 1fr 1fr; }
    }
```
Replace with:
```css
    @media print {
      .masthead, .tabs, #page-upload, #page-mapping,
      .top-bar, #date-filter, #unmapped-notice,
      #pdf-btn { display: none !important; }
      #page-stats { display: block !important; padding: 0; }
      body { background: #fff; }
      .panel { break-inside: avoid; }
      .grid-2 { grid-template-columns: 1fr 1fr; }
    }
```
(The base theme is already light/paper-toned, so the dark→light overrides for `.metric`/`.metrics`/`.bar-track`/text colors are no longer needed — the on-screen colors already print correctly. `body { color: #111 }` is dropped too, since `--ink` at `#1d2326` is already print-appropriate.)

- [ ] **Step 3: Verify**

```bash
cd /Users/t11brunner/Developer/stundentracker
grep -n "@media print" -A 12 index.html
```
Confirm the block matches the replacement above exactly (12 lines including braces).

- [ ] **Step 4: Functional check**

Load a sample `.ics`, go to Statistiken, open the browser's print preview (`Cmd+P` or the `#pdf-btn` → "Als PDF exportieren"). Confirm: masthead/tabs/top-bar/date-filter are hidden, panels/charts/metrics render on a white page without the removed `!important` overrides causing any visual regression (colors should look identical to on-screen, since on-screen is already light).

- [ ] **Step 5: Commit**

```bash
cd /Users/t11brunner/Developer/stundentracker
git add index.html
git commit -m "Simplify print styles now that the base theme is print-ready"
```

---

## Task 10: Final end-to-end verification pass

**Files:** none modified — this task only verifies.

**Interfaces:**
- Consumes: the fully migrated `index.html` from Tasks 1–9.
- Produces: a pass/fail confirmation that the migration is complete and nothing regressed functionally.

- [ ] **Step 1: Full regression grep (must all be zero)**

```bash
cd /Users/t11brunner/Developer/stundentracker
grep -c "var(--bg)\|var(--surface)\|var(--surface2)\|var(--border)\|var(--border2)\|var(--text)\|var(--muted)\|var(--mono)\|var(--sans)\|accent2\|accent3\|DM Mono\|DM Sans\|#c8f060\|#60c8f0\|#f060a0\|#f0c060\|#60f0a0\|#a060f0\|#f09060\|#60f0f0\|#f060f0\|#90f060\|#f0f0ee\|#888884\|#f06060\|rgba(255,255,255\|rgba(200,240,96\|rgba(240,96,96\|rgba(96,200,240\|rgba(240,192,96\|color-scheme: dark\|<header>" index.html
```
Expected: `0`.

- [ ] **Step 2: Start (or confirm running) the local server**

```bash
cd /Users/t11brunner/Developer/stundentracker
curl -sf http://localhost:8765/index.html >/dev/null || (python3 -m http.server 8765 >/tmp/stundentracker-server.log 2>&1 &)
sleep 1
curl -sf http://localhost:8765/index.html >/dev/null && echo "server OK"
```

- [ ] **Step 3: Full click-through in the browser**

Using the Browser preview tool, navigate to `http://localhost:8765/index.html` and walk through:
1. Screenshot the upload tab (masthead, pill nav, drop zone, folder panel, how-to).
2. Read console messages — expect zero errors.
3. Load `ETH Präsenz.ics` (or `ETH Selbststudium.ics`) from the repo root via the drop zone / file picker.
4. Confirm it auto-switches to the Statistiken tab and renders: metric tiles, Belastung panel with a colored score, subject bars, doughnut chart, month/week/day charts, weekday/hour bars.
5. Click "Fach-Mapping" — confirm the pill nav shows the active state, page fades in, add a throwaway rule (e.g. pattern `Test`, subject `test`), confirm it appears as a blue pill in the table, then delete it via "entfernen".
6. Go back to Statistiken, use the date-filter inputs (native date picker should render in light mode, not dark), click "Anwenden" then "Zurücksetzen".
7. Open print preview (`Cmd+P`) — confirm chrome is hidden and the stats content is present on a white page.
8. Read console messages again after all interactions — expect zero errors.

- [ ] **Step 4: Stop the server**

```bash
pkill -f "http.server 8765" || true
```

- [ ] **Step 5: Confirm `ta-website/` was never touched**

```bash
cd /Users/t11brunner/Developer/ta-website
git status --porcelain
```
Expected: empty output (no changes at all in that repo).

- [ ] **Step 6: Final commit (only if Step 3 surfaced small fixups)**

If Step 3 found any visual bug requiring a fix, apply it, re-verify, and commit with a message describing exactly what was fixed. If everything passed clean, no commit is needed for this task — the migration is complete as of Task 9's commit.
