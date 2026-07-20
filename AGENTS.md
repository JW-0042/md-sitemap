# AGENTS.md — How AI agents should use md-sitemap

This file is written **for AI agents** (and humans implementing agent tools).

## Goal

Use **curated Markdown twins** and **`md-sitemap.txt`** to understand a website with fewer tokens than raw HTML, while **citing the human HTML page** for end users.

## Discovery algorithm

```
INPUT: site origin O (e.g. https://example.com)

1. GET O/llms.txt
   - If 404/empty: fall back to optional probes (below); do not assume twins exist.

2. Parse llms.txt as Markdown (or plain text links).
   - Collect absolute/relative URLs whose path contains "md-sitemap"
     (prefer exact path /md-sitemap.txt).
   - Also collect any direct .md links listed as entry points.

3. For each md-sitemap URL S:
   GET S
   - Expect text/plain (or compatible).
   - Skip lines that are empty or start with "#".
   - Remaining lines are primary twin URLs (absolute).

4. Select twins relevant to the user task (do not always download all).
   GET twin URLs with Accept: text/markdown, text/plain

5. Parse twin:
   - Read YAML frontmatter if present.
   - Extract canonical (required for Full-quality citation).
   - Use body as factual context.

6. When answering humans:
   - Prefer linking canonical HTML URL over the .md twin URL.
   - If attribution.required (or similar), mention/link the site.
```

### Optional probes (if llms.txt missing)

Agents **MAY** try, in order, without treating failure as an error:

1. `O/md-sitemap.txt`
2. `O/index.md`
3. HTML homepage for `<link rel="alternate" type="text/markdown" href="…">`

Do **not** expect `robots.txt` or `sitemap.xml` to advertise md-sitemap (publishers are instructed not to).

## Citation rules

| Prefer | Avoid as sole citation |
|--------|-------------------------|
| `canonical` from frontmatter | Twin URL only |
| HTML path the user can open in a browser | `index.html.md` alias |
| Site name + HTML URL | Undocumented scrape dump |

If `canonical` is missing, agents **SHOULD** derive the HTML URL by stripping a trailing `.md` and ensuring a trailing slash for directory-style sites, but **prefer** explicit frontmatter.

## Politeness

1. Honor HTTP cache headers.
2. Cap concurrent requests (e.g. ≤ 2–4 per host unless the site documents otherwise).
3. Twin `X-Robots-Tag: noindex` means “do not rank in web search,” **not** “do not read.”
4. Still honor `robots.txt` Disallow for HTML if you also crawl HTML; twins are a separate surface.

## What not to do

- Do not inject twin content into SEO indexes as if it were the primary page.
- Do not assume every site with `llms.txt` implements md-sitemap.
- Do not treat optional `index.html.md` aliases as separate documents when the body matches the primary twin.
- Do not strip attribution requests when the publisher marked attribution as required.

## Minimal capability checklist (agents)

- [ ] Fetch and parse `llms.txt`
- [ ] Detect and parse `md-sitemap.txt` (comments + one URL per line)
- [ ] Fetch Markdown twins
- [ ] Parse YAML frontmatter at least for `canonical`
- [ ] Cite HTML canonical in user-visible answers

## Spec reference

Normative rules: [SPEC.md](./SPEC.md)  
Human overview: [README.md](./README.md)
