# /add-app

Scaffold a new app marketing page and home page card from a MARKETING.md source file.

## Usage

```
/add-app <DisplayName> <path/to/MARKETING.md>
```

Examples:
```
/add-app "Budget Tracker" ~/Workspace/BudgetTracker/MARKETING.md
/add-app Journaly ~/Workspace/Journaly/MARKETING.md
```

## What this skill does

1. Reads the MARKETING.md source file.
2. Derives naming conventions from the display name.
3. Creates a new detail page (`<slug>.html`) modeled on the existing app pages.
4. Adds a new app card to `index.html`.
5. Registers the new app in `.claude/commands/update-app-pages.md` so future `/update-app-pages` runs include it.
6. Reports what was created and what still needs to be done manually (icon, App Store URL).

## Naming conventions

Derive these from the display name:

| Variable | Rule | Example (display name "Budget Tracker") |
|---|---|---|
| `slug` | lowercase, remove spaces and special chars | `budgettracker` |
| `css-prefix` | first letter of each word, lowercase, max 3 chars | `bt` |
| `icon-path` | `<slug>/icon.png` | `budgettracker/icon.png` |
| `html-file` | `<slug>.html` | `budgettracker.html` |

## Building the detail page

Model the page on `scorepad.html` and `retirementplanner.html`. Preserve all structural patterns exactly:

### Head
```html
<meta name="description" content="<AppName> — <tagline from MARKETING.md>. <platforms>." />
<title><AppName> | Easy Chair Apps Lab</title>
<link rel="stylesheet" href="styles.css" />
<style>
  /* =====================================================
     <APPNAME UPPERCASE> PAGE
     ===================================================== */
  ...all page-scoped CSS using <css-prefix>- namespace...
</style>
```

### CSS
Copy the full CSS block from `scorepad.html`, replacing every `sp-` prefix with `<css-prefix>-`. Adjust the hero `::before` gradient colors only if MARKETING.md mentions a strong brand color — otherwise keep the default violet/teal gradient.

### Header and footer
Use the identical `<header class="legal-header">` and `<footer>` from any existing page — do not modify them.

### Hero section
- App icon: `<img src="<icon-path>" alt="<AppName> app icon" />`
- Title: from MARKETING.md App Identity `App name` field
- Tagline: from MARKETING.md tagline (italic serif, use the best one if multiple are listed)
- Sub-tagline: one sentence description from MARKETING.md Product Summary
- Platform badges: derive from MARKETING.md `Platforms` field — render one `<css-prefix>-platform-badge` span per platform (iPhone, iPad, Mac, etc.) — omit if not listed
- App Store button: use the identical Apple SVG and button structure from existing pages; keep `href="#"` and the "Coming soon" note unless MARKETING.md has a real App Store URL

### Feature cards
- Use a 3-column grid (same `.sp-features` / `.rp-features` pattern)
- Derive cards from the Key Features section of MARKETING.md
- Aim for 6, 9, or 12 cards (multiples of 3) for a clean grid; combine minor features or split large ones to hit a multiple of 3
- Assign an appropriate SVG line icon (24×24, stroke="white") to each card — pick from the icon vocabulary already used in other pages, or use a new Feather-icons-compatible path
- Keep card titles short (2–4 words); descriptions 1–2 sentences

### Free vs Pro section
Include this section only if MARKETING.md has a pricing model. Derive tier names, feature lists, and pricing note from MARKETING.md. If no pricing info exists, omit the section entirely.

### Why [AppName]? section
Include if MARKETING.md has a differentiators or "Why" list. Use the numbered `<css-prefix>-why-item` pattern. If MARKETING.md does not have an explicit list, derive 3–5 items from the product summary and key features.

### Privacy callout
Always include. Use the headline and body text from MARKETING.md's Privacy section if present; otherwise write one based on the on-device / no-account / no-analytics facts stated in the Markdown.

### Disclaimer
Include only if the app deals with financial, medical, or legal subject matter. Model on the `rp-disclaimer` block in `retirementplanner.html`.

### Tip jar / pricing note
Include a brief tip jar line (like ScorePad's) only if MARKETING.md mentions a free app with a tip jar.

## Adding the index.html card

Add a new `<article class="app-card">` inside the `<div class="app-grid">` in `index.html`, after the last existing card:

```html
<article class="app-card">
  <div class="app-icon" style="background:none;overflow:hidden;">
    <img src="<icon-path>" alt="<AppName> app icon" style="width:100%;height:100%;object-fit:cover;display:block;" />
  </div>
  <h3><AppName></h3>
  <p><one-sentence description from MARKETING.md></p>
  <a href="<slug>.html" class="app-card-link">Learn more &rarr;</a>
</article>
```

## Registering in update-app-pages.md

Add a row to the source-to-page mapping table in `.claude/commands/update-app-pages.md`:

```markdown
| `<MARKETING.md path>` | `<slug>.html` |
```

## After scaffolding

Report to the user:
- Which files were created or modified
- That the app icon needs to be placed at `<slug>/icon.png` before the page will render correctly
- That the App Store URL in the hero button should be updated in `<slug>.html` once the app is live
- Offer to run `/update-app-pages <slug>` immediately if the icon is already in place

Then ask if the user wants to commit the changes.
