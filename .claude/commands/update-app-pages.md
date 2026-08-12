# /update-app-pages

Rebuild app marketing pages from their source MARKETING.md files.

## Source-to-page mapping

| Source | HTML page |
|--------|-----------|
| `~/Workspace/RetirementPlanner/MARKETING.md` | `retirementplanner.html` |
| `~/Workspace/ScorePad/MARKETING.md` | `scorepad.html` |
| `~/Workspace/HourlyTimeTracker/MARKETING.md` | `hourlytimetracker.html` |
| `~/Workspace/TaxCalculator/MARKETING.md` | `taxcalculator.html` |
| `~/Workspace/HomeKeep/MARKETING.md` | `homekeep.html` |

## Arguments

- No argument: process all apps
- App name (e.g. `retirementplanner` or `scorepad`): process only that app

> To add a new app to this list, use the `/add-app` skill — it creates the detail page, updates `index.html`, and registers the mapping here automatically.

### App version source

Each app's current version is read from `MARKETING_VERSION` in its Xcode project. The project lives in
the same directory as the app's `MARKETING.md` (e.g. `~/Workspace/ScorePad/`). Read it with:

```bash
grep -h "MARKETING_VERSION" <project_dir>/*.xcodeproj/project.pbxproj | head -1
```

Take the first value (all targets share the same marketing version). This version drives the home-page
"What's New" section (see **News section maintenance** below).

## Steps

**First, prune expired news** (runs once per invocation, regardless of which apps are processed — even
if a specific app was named): remove any expired "What's New" entries from `index.html`. See
**News section maintenance -> Pruning expired entries**.

Then, for each app being processed:

1. **Read** the source `MARKETING.md` file.
2. **Read** the current HTML page.
3. **Read the app's current version** (`MARKETING_VERSION`, see **App version source**).
4. **Determine if a rebuild is needed.** Compare the content sections of the HTML against the Markdown. If nothing has changed, skip the rebuild and report the app as up-to-date. (Version handling in the next step still runs even when no rebuild is needed.)
5. **Rebuild the HTML page** from the Markdown content, following the rules below (skip if step 4 found no changes).
6. **Update the home-page news section for a version change.** Compare the current version against the version recorded on the app's home-page card. If it changed, add a "What's New" entry. See **News section maintenance -> Adding an entry on version change**.
7. After all apps are processed, report which pages were rebuilt, which were already current, and which news entries were added or pruned.

## Rebuild rules

### Preserve (never change from existing HTML)

- All `<style>` block CSS — every rule, class name, variable reference, animation
- Page `<head>` structure: charset, viewport, `<link rel="stylesheet">`, title format
- `<header>` and `<footer>` HTML
- App icon `<img>` tag and its container
- App Store button HTML and SVG (the Apple logo path)
- Section scaffold: class names, `<div>` nesting, `::before` pseudo-element patterns
- All SVG feature icons — keep the same icon for each feature card if it still applies; only swap an icon if a feature is entirely new and you must choose one
- `"Coming soon"` note under the App Store button
- Responsive `@media` breakpoints

### Derive from MARKETING.md

- Page `<title>` and `<meta name="description">`
- Hero: app name `<h1>`, tagline, sub-tagline, platform badges (if present in Markdown)
- Feature cards: title, body copy, and (if clearly described) icon choice — use the same 3-column grid layout
- Free/Pro pricing section: tier names, feature lists, pricing note
- "Why [App]?" numbered list: heading and body copy for each item
- Privacy callout: headline and body paragraph
- Disclaimer box (Retirement Planner only): body copy
- Tip jar line (ScorePad only): text content

### Content mapping guidance

- If MARKETING.md adds a new section not currently in the HTML, add it using the same card/section pattern already in the file.
- If MARKETING.md removes a section present in the HTML, remove it.
- If a section is renamed or reordered in the Markdown, apply the same change in the HTML.
- Preserve HTML entity encoding (`&mdash;`, `&amp;`, `&rarr;`, `&ndash;`) — do not use raw Unicode dashes or ampersands in HTML content.

## News section maintenance

The home page (`index.html`) has a "What's New" section — `<section class="whats-new">` containing a
`<ul class="whats-new-list">` of `<li class="update" data-date="YYYY-MM-DD">` entries. Each entry links
to an app page and names the app + version. A client-side script *hides* entries older than
`MAX_AGE_DAYS` (currently **30** days); this skill additionally **removes** expired entries from the
markup so the file doesn't accumulate stale history.

### Version record on each app card

Each app's last-published version is stored as a `data-version` attribute on its
`<article class="app-card">` in `index.html`, e.g.:

```html
<article class="app-card" data-version="1.2.0">
```

This is the persistent record of "what version the site last announced" — it does **not** expire, so
version-change detection keeps working after news entries are pruned.

### Adding an entry on version change

For the app being processed:

1. Read the current version (`MARKETING_VERSION`) and the `data-version` on the app's card.
2. **If the card has no `data-version` yet** (first run since this feature was added), set
   `data-version` to the current version and add **no** news entry — this only establishes a baseline.
3. **If `data-version` differs from the current version**, a version change occurred:
   - Update the card's `data-version` to the current version.
   - **Remove any existing news entry for this same app** (any `<li class="update">` whose link points
     to this app's `<slug>.html`). An app should appear in the list at most once — the newest version
     replaces the old entry rather than stacking beside it.
   - Insert a new entry at the **top** of `<ul class="whats-new-list">` (newest first), using today's
     date (`date +%F`) as `data-date`:

     ```html
     <li class="update" data-date="YYYY-MM-DD">
       <a href="<slug>.html" class="update-link">
         <span class="update-badge">New</span>
         <span class="update-text"><strong><AppName> <version></strong></span>
       </a>
     </li>
     ```
   - **Keep it to just the app name and version — no description.** The entry is only a hint that a new
     version is available; the app page has the details. Do not add a summary blurb (long text wraps
     awkwardly on the card).
4. **If `data-version` equals the current version**, do nothing to the news section for this app.

### Pruning expired entries

Runs once per invocation, before processing apps:

1. Get today's date: `date +%F`.
2. For each `<li class="update" data-date="...">` in `index.html`, compute its age in days from today.
3. **Remove** (delete the entire `<li>`) any entry older than **30** days (matching `MAX_AGE_DAYS` in
   the page script). Keep the threshold in sync with that constant.
4. Leave the `<section>`, `<ul>`, and the client-side script intact even if the list becomes empty — the
   script already hides an empty section.

## After rebuilding

Ask the user if they want to commit the changes (including any `index.html` news/version updates). If
yes, use the `commit-commands:commit` skill to commit. Use message
`feat: rebuild [app name] page from MARKETING.md` when a page was rebuilt, or
`chore: update home-page news section` when only the news section changed.
