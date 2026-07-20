# Changelog

## 0.1.1 — 2026-07-20

Documentation and usability polish from engineering review (Core/Full intent unchanged).

- README: punchier intro, scannable quickstart, curl smoke test, adopters table, logo, how-to-adopt.
- SPEC: primary twin mapping clarified as normative; recommended `title` and `schemaVersion`; multilingual §3.6; security/budget notes; schema link.
- AGENTS: graceful degradation, fetch budgets, abuse resistance.
- IMPLEMENTATION: ETag/Last-Modified guidance, multilingual sister-site notes, stack adoption table.
- Primary schema path: `schemas/frontmatter.schema.json` (legacy alias retained).
- CONTRIBUTING.md, assets/md-sitemap-logo.svg.
- Examples use shorter md-sitemap comments and richer frontmatter.

## 0.1.0 — 2026-07-20

- First public draft of the **Markdown Sitemap & Page Twin Protocol**.
- Documentation completed with **Grok 4.5 (high)** via **Grok Build** (xAI).
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
