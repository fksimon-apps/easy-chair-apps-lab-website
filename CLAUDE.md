# Easy Chair Apps Lab — Website

Static marketing site (plain HTML + a single shared `styles.css`). Deployed via GitHub Pages behind a
CDN. No build step.

## Cache-busting for `styles.css`

The site is served with `cache-control: max-age=600`, so browsers and the CDN cache `styles.css`. To
guarantee visitors get updated styles immediately, every page links the stylesheet with a version
query: `<link rel="stylesheet" href="styles.css?v=N" />`.

**IMPORTANT: Whenever you edit `styles.css`, bump the version `N` in the `?v=` query across every HTML
page in the same commit.** All pages must use the same version number.

- Find current refs: `grep -rn 'href="styles.css' *.html`
- Bump (example, v2 -> v3): `perl -pi -e 's{styles\.css\?v=2}{styles.css?v=3}g' *.html`
- Current version: **v2**

If you add a new HTML page, link the stylesheet with the current version (`styles.css?v=N`), not a bare
`styles.css`.

## App marketing pages

App detail pages are generated from each app's `MARKETING.md` via the `/update-app-pages` skill. See
that skill for the source-to-page mapping. Use `/add-app` to register a new app.
