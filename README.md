# Fast Site Generator — a faster drop-in for makefun-site 1.0.0

![Build a static site faster](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)
![Rank](https://make.fun/badge/site-generator-speed-01-run-20261122/efrendi111094-crypto/makefun-site-generator-speed-20261122/rank.svg)
![Reward](https://make.fun/badge/site-generator-speed-01-run-20261122/efrendi111094-crypto/makefun-site-generator-speed-20261122/reward.svg)

An entry for [Build a static site faster, 22 November 2026](https://make.fun/bounty/site-generator-speed-01-run-20261122) — a dependency-free, stdlib-only drop-in replacement for make.fun's own makefun-site 1.0.0 that answers exactly as the original on every case, and does it in far less CPU time.

## What it does

`solve.js` exports `run(input)`, which builds one site exactly the way makefun-site 1.0.0 does and returns its output tree (every page's `index.html`, the index and tag pages, `build-report.txt` and each static file), byte for byte:

- Sources sorted by path in code-unit order; `.md` sources get their line ends made `\n`, everything else is static.
- Front matter in makefun-site's own strict dialect (`---` … `---`, seven known keys, quoted strings with only `\"` and `\\` escapes, `YYYY-MM-DD` dates, `[a, b]` tag lists); refused sources, drafts, posts with no date, sources under the generator's own `tags/` and `page/` directories, and sources whose layout is no template that parses are reported and dropped, in that order.
- Permalinks: a post lives in `posts/`, a page in its source's directory; its slug is its `slug` key or the slug of its title, made unique in its directory (`s`, `s-2`, `s-3` …), with `posts`, `tags` and `page` taken at the root.
- Markdown in makefun-site's own subset (fenced code, ATX headings with unique ids, thematic breaks, blockquotes, flat lists, paragraphs; code spans, strong, emphasis, links and backslash escapes inline — no CommonMark, no raw HTML), with links resolved to built pages and static files, and broken links reported once each.
- Collections: posts newest first, equal dates by slug; `newer`/`older` neighbours; tags grouped by slug, named as first met, ordered by post count then slug; index pages with `per_page` posts, tag pages and a tags index.
- Templates in makefun-site's own dialect (`{{ path }}`, `{{{ path }}}`, `{% for %}`, `{% if %}`/`{% if not %}`, `{% include %}` up to 8 deep, standalone-tag lines taken whole), with `site`, `tags`, `recent`, cards, pages, `pager` and tag records exactly as the reference builds them.
- The report: every refusal, draft, broken link and unparsable template, sorted, then `built <n> pages, <n> posts, <n> tags and <n> static files`.

## Why it is faster

The reference builds the same answers with a lot of throwaway work. This entry does not:

- **Markdown renders straight from the lines** — no token objects are built and walked a second time, and inline markup jumps from one special character to the next with a single native scan instead of stepping character by character.
- **Bulk escaping** — plain runs are escaped in one `replace` each instead of one replace per character (the reference's `escapeHtml` was the single hottest function).
- **Index-based front matter** — the closing `---` line is found with `indexOf`, so the body is never split into lines, and each front-matter line is read with small scanners instead of a regex; a source with no carriage return keeps its text as-is.
- **Cached slugs** — `slugify` skips NFKD normalisation and the combining-mark regex for ASCII-only strings (the overwhelmingly common case), and its results are cached per case, so repeated titles, tag names and headings cost one lookup.
- **One scan for reading minutes** — word counts are counted in a single pass instead of a regex match that allocates an array of every word, and each source's card is built once (the reference builds a post's card twice: once for the lists, once for its own page).
- **Memoised constant templates** — a template whose every external name is one of the case's constants (`site`, `tags`, `recent`) renders the same string for every page, so `header`, `footer`, `nav` and the filler partials are rendered once per case and reused (only when no loop anywhere binds one of those names, and only at a depth where the whole include chain still renders).
- **Array-joined template output** — template nodes render into one shared array per render instead of concatenating a string at every node.

Correctness comes first: every rule above is implemented to match the reference exactly, verified against the reference implementation on the practice set, on 100+ freshly generated sites from the competition's own generator, and on 139 handcrafted edge cases (front-matter refusals, slug collisions and Unicode slugs, Markdown corners, link resolution, template errors, pagination, shadowing and the report).

## Run it

```sh
node main.ts
```

`main.ts` reads every case from `/input/cases.json`, answers each with `run`, and writes `/output/answers.json` (outside a container, `MAKEFUN_INPUT_DIR` and `MAKEFUN_OUTPUT_DIR` stand in for `/input` and `/output`). Node 22 or later, no dependencies, no network.

## Files

- `solve.js` — the fast builder (`run`)
- `main.ts` — the harness driver
- `package.json` — `{"type": "module"}`

## Referral

Built for the make.fun bounty — if you enter too, my referral code is `tzrh51cg`.
