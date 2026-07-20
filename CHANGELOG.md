# Changelog

## 0.1.0 — 2026-07-20

- First public draft of the **Markdown Sitemap & Page Twin Protocol**.
- Defines:
  - Page twin URL mapping (`/path/` → `/path.md`, homepage → `/index.md`)
  - Optional `index.html.md` aliases
  - Twin frontmatter (`canonical` required)
  - Attribution guidance
  - `/md-sitemap.txt` format and discovery via `llms.txt` only
  - SEO isolation (`X-Robots-Tag: noindex`, no SEO sitemap listing)
  - Conformance levels **Core** and **Full**
- Adds agent discovery algorithm (`AGENTS.md`), implementation recipes, examples, and frontmatter JSON Schema.
- Reference production surfaces: meninydnes.site, svatek-dnes.cz.
