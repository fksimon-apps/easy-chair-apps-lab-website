# /update-app-pages

Rebuild app marketing pages from their source MARKETING.md files.

## Source-to-page mapping

| Source | HTML page |
|--------|-----------|
| `~/Workspace/RetirementPlanner/MARKETING.md` | `retirementplanner.html` |
| `~/Workspace/ScorePad/MARKETING.md` | `scorepad.html` |

## Arguments

- No argument: process all apps
- App name (e.g. `retirementplanner` or `scorepad`): process only that app

## Steps

For each app being processed:

1. **Read** the source `MARKETING.md` file.
2. **Read** the current HTML page.
3. **Determine if a rebuild is needed.** Compare the content sections of the HTML against the Markdown. If nothing has changed, skip that app and report it as up-to-date.
4. **Rebuild the HTML page** from the Markdown content, following the rules below.
5. After all pages are processed, report which were updated and which were already current.

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

## After rebuilding

Ask the user if they want to commit the changes. If yes, use the `commit-commands:commit` skill to commit with message:
`feat: rebuild [app name] page from MARKETING.md`
