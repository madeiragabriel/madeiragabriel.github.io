# Website principles and architecture

A playbook for whoever maintains this site next, human or AI. It records what was built, why, and the decisions that must not be undone silently. Read `UPDATING.md` first for routine content changes.

## Stack

- Hugo extended (0.152.x locally and in CI; minimum 0.148.2 required by the theme).
- Hugo Blox `blox-tailwind` v0.10.0, imported as a Hugo module and vendored in `_vendor/` (committed). The theme is used for its head partial, Tailwind pipeline, Alpine.js, and the search-modal scaffolding; almost every visible template is overridden in `layouts/`.
- Tailwind CSS v4 is compiled by Hugo's `css.TailwindCSS`, which needs `tailwindcss`, `@tailwindcss/cli` and `@tailwindcss/typography` from `package.json` installed (`npm install`). CI runs this step.
- Pagefind builds a static search index after Hugo (`npx pagefind --site public`).
- GitHub Pages user site, deployed by `.github/workflows/deploy.yml` on every push to `main`. `baseURL` is set in `hugo.yaml` (`https://madeiragabriel.github.io/`) and is never derived from CI variables.

## Layout system (Hugo 0.146+ conventions)

Hugo's new layout tree is used: `layouts/baseof.html`, `layouts/_partials/`, `layouts/_shortcodes/`. Overrides of Blox partials go in `layouts/_partials/<same path>`; `layouts/partials/` would be ignored.

| File | Role |
|---|---|
| `layouts/baseof.html` | Page shell: skip link, sticky header, `<main data-pagefind-body>`, footer. |
| `layouts/_partials/components/headers/navbar.html` | Brand = owner name (uppercase) linking home; links right; search as icon only; CSS-only hamburger at <1024px. |
| `layouts/_partials/components/search-modal.html` | Blox modal patched: `import('{{ "pagefind/pagefind.js" | relURL }}')`, Google `site:` fallback when Pagefind fails, no vendor link. |
| `layouts/_partials/site_footer.html` | Dark footer, site nav, opt-out mysite credit (`params.mysite.credit`), rendered exactly once. |
| `layouts/_partials/hooks/head-end/mysite-meta.html` | Generator meta + homepage Person JSON-LD (both gated by `params.mysite.discovery`), favicon links, and the scholarly-metadata partial on publication/talk pages. |
| `layouts/_partials/scholarly_meta.html` | Google Scholar `citation_*` tags and ScholarlyArticle/Book/CreativeWork JSON-LD from front matter. |
| `layouts/_partials/related_finder.html` | Automatic "See also" (algorithm below). |
| `layouts/_partials/citation_row.html` | One dense citation line; used by the Research page, homepage accordions and talk/software lists. Carries `data-tab`, `data-year`, `data-slug`, `data-tags`. |
| `layouts/_partials/functions/pub_tab.html` | Maps a page to its Research tab id (honours `data/writings_legacy_map.json` if present). |
| `layouts/landing/list.html` | Homepage: hero (photo/monogram left, text right, stacked ≤640px) and research-area `<details>` accordions with 4 featured items and a "View all" link to the filtered Research page. |
| `layouts/publication/list.html` | The Research page: tabs, sidebar filters, sort, live count, BibTeX export, hash routing. All entries server-rendered; JS only filters. |
| `layouts/publication/single.html` | Publication page: title, authors, venue/date, link buttons, featured image, abstract, tags, See also. No publication-type badge. |
| `layouts/authors/list.html` | People page, Presentation B: names-only alphabetized list, links from `data/coauthors.json`. |
| `layouts/_partials/prose_page.html` + `page/`, `single.html`, `list.html` | Generic prose pages (bio, teaching, resources, contact). |
| `layouts/404.html`, `layouts/robots.txt`, `layouts/_shortcodes/staticrel.html` | Not-found page with search, crawler policy, subpath-safe URL shortcode. |

## Data files

- `data/site.yaml`: owner identity and homepage intro. Single source for name, role, institution, email, initials.
- `data/research_areas.json`: areas with `id`, `name`, `description`, `tags`. A publication belongs to an area when any of its tags matches; this feeds the homepage accordions and the area filter.
- `data/coauthors.json`: name → `{url, type}` or `null`. Auto-discovered by web search at build time; must be reviewed by the owner.
- `data/featured_publications.yaml`: slugs of working papers to spotlight (reserved for a future homepage block).
- `i18n/en.yaml`: every button and label string.

## Content model

Each publication is `content/publication/<slug>/index.md`; the directory name is the permanent URL (`permalinks: /publication/:contentbasename/`). Front matter: `title`, `date`, `authors`, `publication_types`, `publication`, optional `abstract`, `links` (`name` + `url`, `url` without leading slash), `tags`, `status` (`working_paper` | `in_progress`), `forthcoming`, `hide_date`, `private`, `dataverse_url`, `dataverse_name`, `related_*`, `see_also`.

Talks and software use the same shape in `content/talk/` and `content/software/`; their templates exist but the sections currently have no content and no menu entry.

Taxonomies `taxonomy` and `term` kinds are disabled (`disableKinds`) so no raw tag or author term pages are generated; People is a dedicated section template.

## "See also" algorithm (`related_finder.html`)

1. Explicit front matter wins: `related_papers`, `related_talks`, `related_software`, `related_datasets`, singular `related_paper` / `related_dataset`, and `see_also` entries with `slug` (score 1000).
2. `see_also` entries with `url` are appended as external `[Link]` rows; `dataverse_url` (or a Harvard Dataverse URL inside the abstract) becomes a `[Dataset]` row.
3. Every other publication/talk/software page is scored: +2 per shared title token (lowercased, stop words stripped, tokens <3 chars dropped; English, Portuguese and Spanish stop words), +1 per shared co-author (exact or surname match, owner excluded), +2 per shared tag.
4. Threshold 4, relaxed to 2 when fewer than 3 explicit picks. Response/comment papers pin their subject; book editions cross-link.
5. Cap 8, sorted by score then year; titles deduplicated after normalization; label chosen from `publication_types` before section.

No script generates a related map; everything is computed at build time.

## Research page behaviour

Tab ids: `all`, `articles`, `chapters`, `working-papers`, `in-progress`, `presentations`, `software`, `data`; only tabs with content render. Filters: keyword (title, authors, venue, abstract, tags; accent-insensitive), research area (by tag membership), year checkboxes, sort. State is mirrored into `location.hash` (`#working-papers&q=covid&years=2022,2021&sort=oldest`) so views are shareable; homepage "View all" links use `#area=<id>`. BibTeX export serializes the visible rows.

## Styling

`assets/css/custom.css` is the only project stylesheet. Palette tokens are CSS custom properties: a greyish olive green family (pure white ground, dark olive-charcoal text, olive accent for buttons and active states, deeper olive for hovers and rules; the favicon uses the same accent). Light mode is forced; the Blox theme toggle is hidden. Fonts are the native system stack. Breakpoints: hero stacks at 640px, filter sidebar stacks at 860px, hamburger below 1024px. `prefers-reduced-motion` strips transitions.

## URL safety

Project-site subpaths are no longer in play (this is a user site at the domain root), but every template still uses `relURL` without a leading slash and content uses `{{</* staticrel "files/x.pdf" */>}}`, so the site would survive a move back under a subpath by changing `baseURL` alone.

## Search and crawlers

`<main data-pagefind-body>` limits the index to page content; breadcrumbs and See also blocks carry `data-pagefind-ignore`. `layouts/robots.txt` allows all crawlers and points to the sitemap and `llms.txt`. `static/llms.txt` lists the key URLs and must be edited by hand if the address changes.

## Decisions to preserve

- Homepage intro (`data/site.yaml`) and bio (`content/bio/`) are different texts. Keep them that way.
- The People page never shows link-type labels or publication counts.
- No publication-type badge on single pages.
- The mysite credit appears once, in the footer, and is opt-out via `params.mysite.credit`; the generator meta and Person JSON-LD are opt-out via `params.mysite.discovery`.
- The favicon is the owner's initials on the accent colour (`static/favicon.ico`, `favicon-32x32.png`, `apple-touch-icon.png`, `assets/media/icon.png`), regenerate all four together if the colour changes.
- Content-flagging comments (`<!-- ... -->`) are the only place for notes to the owner; nothing process-related ever renders.

## Known gaps at handoff

- No portrait yet (monogram placeholder).
- Six co-authors have no verified link; all others were guessed by search and need review.
- Working papers carry no abstracts or PDFs yet; add `abstract:` and `links:` when available.
- "Green Rhetoric, Anti-Environmental Voting" is now a journal article (Environmental Politics, online first); add volume, issue, pages and abstract when available and confirm the author list.
- `content/talk/` and `content/software/` sections are empty and hidden from the menu.
