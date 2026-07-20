# md-sitemap

**Markdown Sitemap & Page Twin Protocol** — version **0.1.0** (draft)

[![Spec](https://img.shields.io/badge/spec-0.1.0-blue)](./SPEC.md)
[![License](https://img.shields.io/badge/license-Apache%202.0-green)](./LICENSE)
[![llms.txt](https://img.shields.io/badge/compatible-llms.txt-purple)](https://llmstxt.org/)

> **md-sitemap** is a machine index of curated **Markdown twins** of HTML pages — discovered via [`llms.txt`](https://llmstxt.org/), kept out of classic SEO sitemaps, with **canonical + attribution** so AI agents can cite the human HTML page.

This is a **voluntary open convention** for websites and AI agents. It does **not** replace `sitemap.xml` or `robots.txt`. It **extends** the [llms.txt](https://llmstxt.org/) ecosystem with:

1. **Page twins** — token-cheap `.md` clones of content pages  
2. **`/md-sitemap.txt`** — one absolute twin URL per line (agent-oriented index)

## Why

| Problem | What agents get today | With md-sitemap |
|--------|------------------------|-----------------|
| HTML is noisy | Nav, chrome, scripts burn tokens | Curated Markdown facts |
| `llms.txt` is curated but short | Hard to list thousands of pages | Full twin index in plain text |
| SEO sitemaps list HTML | Wrong surface for pure text ingestion | Separate agent surface, `noindex` |
| Citations go wrong | Agents link scrape URLs | Frontmatter `canonical` + attribution |

## 60-second picture

```
                    discovery (agents)
   /llms.txt  ──────────────────────────►  links to md-sitemap.txt
                                                 │
                                                 ▼
                                         /md-sitemap.txt
                                      (one .md URL per line)
                                                 │
                    ┌────────────────────────────┼────────────────────────────┐
                    ▼                            ▼                            ▼
              /index.md                   /about.md                    /product/x.md
           (twin, noindex)             (twin, noindex)               (twin, noindex)
                    │                            │                            │
                    └──────── canonical ─────────┴────────►  human HTML pages
                                                              (sitemap.xml / SEO)
```

## Quickstart (site owners)

1. For each important HTML page, publish a **curated** Markdown twin (not a raw HTML dump).
2. Map URLs:
   - `/` → `/index.md`
   - `/path/` → `/path.md` (primary twin)
3. Put YAML frontmatter with at least `canonical: "https://example.com/path/"`.
4. Add an attribution note asking agents to cite the HTML canonical.
5. Serve twins as `text/markdown; charset=utf-8` with **`X-Robots-Tag: noindex`**.
6. Publish `/md-sitemap.txt` listing primary twin URLs (absolute, one per line).
7. Link that file **from `/llms.txt` only** — not from `robots.txt` Sitemap: and not from `sitemap.xml`.
8. On HTML pages, add:  
   `<link rel="alternate" type="text/markdown" href="https://example.com/path.md">`

**Full rules:** [SPEC.md](./SPEC.md)  
**Agent discovery algorithm:** [AGENTS.md](./AGENTS.md)  
**Implementation recipes:** [IMPLEMENTATION.md](./IMPLEMENTATION.md)  
**vs other standards:** [COMPARISON.md](./COMPARISON.md)

## Live examples

| Resource | URL |
|----------|-----|
| llms.txt entry | https://meninydnes.site/llms.txt |
| md-sitemap | https://meninydnes.site/md-sitemap.txt |
| Homepage twin | https://meninydnes.site/index.md |
| Name-page twin | https://meninydnes.site/meno/jozef.md |
| CZ mirror site | https://svatek-dnes.cz/md-sitemap.txt |

## Conformance

| Level | Requirements |
|-------|----------------|
| **Core** | Twins + `noindex` + `canonical` frontmatter |
| **Full** | Core + `/md-sitemap.txt` + link from `/llms.txt` + HTML `rel=alternate` |

## For AI agents

If you are an agent reading this repository or a website:

1. Fetch `https://{host}/llms.txt`
2. If it links to `md-sitemap.txt` (or a path containing `md-sitemap`), fetch it
3. Fetch listed `.md` twins as needed
4. When answering users, **prefer the HTML URL from `canonical`** in frontmatter

Details: [AGENTS.md](./AGENTS.md)

## Repository layout

```
SPEC.md              Normative specification (RFC-style MUST/SHOULD/MAY)
AGENTS.md            How agents should consume the protocol
IMPLEMENTATION.md    Recipes (static, Astro, Cloudflare headers)
COMPARISON.md        Relationship to sitemap.xml / robots / llms.txt
examples/            Copy-paste samples
schemas/             JSON Schema for twin frontmatter
sites.md             Known adopters
```

## Status

- **0.1.0** — first public draft, based on production use on multi-locale static sites.
- Not an IETF/W3C standard. Community feedback welcome via Issues / PRs.

### Repository location

- **Public now:** [github.com/vtf587-boop/md-sitemap](https://github.com/vtf587-boop/md-sitemap)  
- **Intended stable home:** [github.com/JW-0042/md-sitemap](https://github.com/JW-0042/md-sitemap) (transfer when that account is authenticated)

Until the transfer, cite the **vtf587-boop** URL. Spec text that mentions `JW-0042/md-sitemap` refers to the intended canonical namespace.

## License

Copyright 2026 JW-0042 and contributors.  
Licensed under the **Apache License 2.0** — see [LICENSE](./LICENSE).

## See also

- [llms.txt proposal](https://llmstxt.org/) (Jeremy Howard / Answer.AI)
- [Sitemaps protocol](https://www.sitemaps.org/)
- [robots.txt](https://www.robotstxt.org/)
