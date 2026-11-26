# Page & Hierarchy Theming (POC)

How this project gives different pages, and whole branches of the AEM page tree,
different looks. Built as a reusable pattern for the Jackson crosswalk/EDS migration.

## The three levels

| Level | Authoring | Resulting markup | Styled by |
|---|---|---|---|
| Page theme | Page Properties → **Theme** (`models/_page.json`) | `<meta name="theme" content="jackson">` → `<body class="jackson">` | `body.<theme>` token block in `styles/styles.css` |
| Hierarchy theme | Bulk metadata sheet `/content/page-styling/metadata`, row `URL: /jackson/**`, `theme: jackson` | Same `<meta name="theme">` on every matching page | Same as above |
| Inline / section style | Section → **Style** (`models/_section.json`) | `<div class="section dark">` | `main .section.<style>` rules in `styles/styles.css` |

`decorateTemplateAndTheme()` in `scripts/aem.js` turns the `theme` (and `template`)
metadata into body classes. No custom JS is needed.

**Precedence:** a theme set in a page's own properties overrides the bulk metadata row.
Leaving it on *Default* (empty) lets the page inherit from the sheet.

## Why tokens

Each theme only re-defines CSS custom properties (`--link-color`, `--background-color`,
`--heading-font-family`, `--surface-*` …). Blocks, header and footer already use these
tokens, so they follow the theme automatically. Section styles re-map the same tokens
locally, so a "Dark" section looks different on a Corporate page than on a Jackson page.
Nothing extra is loaded, so performance is unaffected.

## Adding a new theme (e.g. for a Jackson sub-brand)

1. Add an option to the `theme` select in `models/_page.json` (value = class name).
2. Add a `body.<value> { … }` token block in `styles/styles.css`.
3. Run `npm run build:json` and `npm run lint`, commit and merge.
4. Optionally add a row in the bulk metadata sheet for the matching AEM subtree.

## Adding a new section style

1. Add an option to the `style` multiselect in `models/_section.json`.
2. Add `main .section.<value>` rules in `styles/styles.css` (prefer re-mapping tokens).
3. Rebuild JSON, lint, commit.

## Required configuration

- `paths.json` maps `/content/page-styling/` to `/` and publishes the bulk metadata
  sheet as `/metadata.json`.
- The site configuration's content source must point at
  `https://<author>/bin/franklin.delivery/<org>/<repo>/main` for **this** repository,
  otherwise the author uses another repository's `paths.json`.
- The bulk metadata sheet must be published (to preview/live) along with the pages.
