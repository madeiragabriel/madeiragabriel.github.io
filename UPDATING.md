# Updating this website

This site is a set of plain-text files. You edit them on GitHub (or in any text editor), and every change pushed to the `main` branch is rebuilt and published automatically within a few minutes. No software needs to be installed to make routine edits.

The live site is https://madeiragabriel.github.io/. The repository is `madeiragabriel/madeiragabriel.github.io`.

## The files you will actually touch

| What you want to change | File |
|---|---|
| Homepage intro, your title, institution, email, initials | `data/site.yaml` |
| Bio, education, appointments, awards | `content/bio/_index.md` |
| A paper, chapter, working paper or project | one folder per item under `content/publication/` |
| Research areas shown on the homepage and in the filter | `data/research_areas.json` |
| Co-author links on the People page | `data/coauthors.json` |
| Courses | `content/teaching/_index.md` |
| Resources for students (Portuguese) | `content/resources/_index.md` |
| Contact details | `content/contact/_index.md` |
| Your C.V. (PDF) | replace `static/files/Gabriel_Madeira_CV.pdf` |
| Your photo | add `assets/media/portrait.jpg` (see below) |
| Button and label wording | `i18n/en.yaml` |
| Menu, site address, footer credit | `hugo.yaml` |

Please review the text in `data/site.yaml` (homepage intro) and `content/bio/_index.md` (bio): both were drafted from your C.V. and should read in your own voice. Keep them different from each other. The homepage intro is a short orientation; the bio is the full narrative.

## Adding a paper

1. Create a new folder under `content/publication/`. The folder name becomes the permanent web address, so choose a short kebab-case slug such as `voter-turnout-brazil-2026` and never rename it afterwards.
2. Inside it, create `index.md` with this front matter (copy an existing one as a template):

```yaml
---
title: "Paper Title"
date: 2026-03-01                # publication date; year is what shows
authors: ["Gabriel Madeira", "Co-Author Name"]
publication_types: ["journal_article"]   # see list below
publication: "Journal Name, 12(3), 100–120 (2026)"
abstract: "One paragraph."
links:
  - name: "Article (PDF)"
    url: "files/paper-slug.pdf"        # no leading slash; file goes in static/files/
  - name: "Publisher's Version"
    url: "https://doi.org/10.xxxx/xxxxx"
tags: ["public opinion", "voting behavior"]
---
```

3. Put the PDF in `static/files/` and reference it as `files/name.pdf` (no leading slash).
4. Optional: drop a `featured.jpg` or `featured.png` next to `index.md` (journal cover, figure) and it appears on the page automatically.

`publication_types` decides which tab the item sits under on the Research page:

| Value | Tab |
|---|---|
| `journal_article` | Articles |
| `book_chapter` or `book` | Book Chapters |
| `report` | Working Papers |
| `report` plus `status: in_progress` | In Progress |
| `presentation` (in `content/talk/`) | Presentations |
| `software` (in `content/software/`) | Software |
| `data` | Datasets |

Other useful fields: `forthcoming: true` adds a "Forthcoming" flag; `hide_date: true` hides the date on the page; `dataverse_url` and `dataverse_name` add a replication-data button; `related_papers: [slug, slug]` pins items in the "See also" list; `private: true` keeps a page out of search engines and the sitemap.

When a working paper is published, edit the same folder: change `publication_types` to `journal_article`, update `publication`, add the DOI link, and remove `status`. Do not create a second folder.

Tags drive two things: the research-area accordions on the homepage and the "Research area" filter. If you add a new tag, also add it to the matching area in `data/research_areas.json` (or it will only be searchable, not filterable by area).

## The People page

The People page is generated automatically from the `authors:` field of every publication: every name except yours becomes a line, alphabetized by surname. You never edit the page itself.

Links are looked up in `data/coauthors.json`, keyed by the exact name string used in the publications. **The links in that file were found by web search during the initial build and are best guesses. Please check every one, correct any that point at the wrong person, and add links for the names that are currently `null`** (Lucas Maia, Marília Vital, Karen Rizzato, Gabriela Peixoto, Yury Machado, Maria Luiza Barretos). Names with `null` appear without a link. The page is not in the menu, but it exists at `/authors/`; add a `People` entry under `menu.main` in `hugo.yaml` to show it. Preferred order: personal website, then university page, then LinkedIn.

If the same person is spelled two ways across papers (for example "Sergio Simoni" and "Sérgio Simoni Júnior"), pick one spelling and use it everywhere; otherwise they appear twice.

## Adding your photo

Save a square portrait (about 720 × 720 px, under 150 KB) as `assets/media/portrait.jpg`. The homepage swaps the monogram for the photo on the next build. Nothing else to change.

## Changing the homepage intro or bio

Intro: `data/site.yaml`, field `intro` (2 to 4 sentences, third person). Bio: `content/bio/_index.md`, ordinary Markdown below the front matter. The C.V. button on the bio page points at `static/files/Gabriel_Madeira_CV.pdf`; replace that file to update the C.V.

## Adding a talk or software

Same structure as a paper, in `content/talk/<slug>/index.md` (with `publication_types: ["presentation"]`) or `content/software/<slug>/index.md`. Create `content/talk/_index.md` with a `title:` the first time you add one so the section page exists. To show it in the menu, add an entry under `menu.main` in `hugo.yaml`.

## Editing on GitHub

Open the file on github.com, click the pencil icon, edit, then "Commit changes" directly to `main`. The Actions tab shows the build; the site updates when the green check appears (usually two to three minutes). If the build fails, the Actions log names the file and line; the most common cause is a missing quotation mark or a stray colon in front matter.

## Site address, footer credit, invisible metadata

`hugo.yaml` holds the public address (`baseURL`, must end with a slash) and the two options from the information form under `params.mysite`: `credit` (the small "Created using GaryKing.org/mysite" line in the footer) and `discovery` (an invisible generator tag and a schema.org Person block for search engines). Set either to `false` to turn it off. If the address ever changes, also update the URLs in `static/llms.txt`.

## Things not to do

- Do not rename or move a folder under `content/` once the site is live; addresses are cited in papers.
- Do not put a leading slash on `url:` values in `links:`.
- Do not edit anything under `_vendor/`; that is the theme, frozen at a known version.
- Do not delete `go.sum`, `package.json` or `_vendor/`; the automated build needs them.
