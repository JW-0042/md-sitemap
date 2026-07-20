# Comparison: md-sitemap vs related conventions

## At a glance

| Criterion | `sitemap.xml` | `robots.txt` | `llms.txt` | **md-sitemap + twins** |
|-----------|---------------|--------------|------------|-------------------------|
| Primary audience | Search engines | All crawlers | AI agents | AI agents |
| Format | XML | Plain directives | Markdown | Plain text index + Markdown pages |
| Scope | Often entire site | Allow/Disallow + SEO sitemap pointer | Curated highlights | Enumerable twin set |
| Page body | No (URLs only) | No | Rarely | **Yes** (twins) |
| SEO indexing | Yes (desired) | Controls crawl | N/A | Twins **noindex** |
| Discovery | robots / GSC | Well-known path | Well-known path | **Via llms.txt** |
| Citation target | HTML URL | — | HTML URL | HTML via `canonical` |

## sitemap.xml

- **Purpose:** Tell search engines which **HTML** URLs to consider.
- **md-sitemap difference:** Lists **Markdown twin** URLs for agents; must stay out of the SEO sitemap to avoid duplicate/thin results.

Use **both**: `sitemap.xml` for Google/Bing; `md-sitemap.txt` for agents.

## robots.txt

- **Purpose:** Crawl budget and policy; optional `Sitemap:` for SEO.
- **md-sitemap difference:** Do **not** advertise `md-sitemap.txt` via `Sitemap:`.
- Twins may still be `Allow`ed; `noindex` is on the HTTP response of the twin.

## llms.txt ([llmstxt.org](https://llmstxt.org/))

- **Purpose:** Curated, human-edited map of high-value resources for LLMs.
- **md-sitemap relationship:** **Complementary.**
  - `llms.txt` = editorial front door (small, intentional).
  - `md-sitemap.txt` = machine-complete index of twins (can be large).
- Sites **SHOULD** keep using llms.txt and **add** a link to md-sitemap under a clear section.

md-sitemap does **not** replace llms.txt. A site with only md-sitemap and no llms.txt is harder for agents that only check llms.txt first.

## Raw HTML scraping

| | HTML scrape | Twin |
|--|-------------|------|
| Tokens | High | Low |
| Structure | DOM noise | Headings + lists |
| Stability | Layout changes break scrapers | Editorial surface |
| Citation | Page URL | Explicit `canonical` |

## “Markdown sitemap” blog posts

Some articles call `llms.txt` itself a “markdown sitemap.” That usage is informal.

**This specification** reserves:

- **`md-sitemap` / `md-sitemap.txt`** for the **one-URL-per-line twin index**, and  
- **page twin** for the per-page `.md` documents with frontmatter and attribution.

If you only publish llms.txt without twins or md-sitemap.txt, you are implementing llms.txt — not Full md-sitemap.

## Optional related files

| File | Role |
|------|------|
| `/llms-full.txt` | Denser prose overview (publisher-defined); **MAY** also link md-sitemap |
| `/meniny.json` / APIs | Structured data; orthogonal to twins |
| `AGENTS.md` (in repos) | Instructions for coding agents; different from this protocol |

## Recommendation

For a public content site that wants both SEO and agent readiness:

1. Keep classic SEO: `robots.txt` + `sitemap.xml` + quality HTML.  
2. Add `llms.txt` (curated).  
3. Add **md-sitemap Full**: twins + `/md-sitemap.txt` + link from llms.txt + `rel=alternate`.
