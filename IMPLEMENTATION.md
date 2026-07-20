# Implementation guide

This document is **non-normative**. Normative rules live in [SPEC.md](./SPEC.md).

## Architecture choices

| Approach | When to use |
| --- | --- |
| **Static files at build time** | SSG sites (Astro, Next export, Hugo, Eleventy) — preferred |
| **Edge/server routes** | Dynamic pages where twins are generated on demand |
| **Hybrid** | SSG for most pages + dynamic for a few |

Production pattern used by reference sites: **generate all twins at build**, deploy as static assets next to HTML, set headers in platform config (`_headers`, `vercel.json`, nginx, etc.).

## How to adopt by stack

| Stack | Pattern |
| --- | --- |
| **Astro** | Endpoint or `getStaticPaths` emitting `.md` + `md-sitemap.txt` (see §7) |
| **Hugo** | Output format or custom output for Markdown twins beside HTML |
| **Eleventy** | Extra template/permalink for `.md` twins in the build graph |
| **Next.js (static export)** | Generate files into `out/` at build, or Route Handlers for twins |
| **Plain static** | Hand-authored or script-generated files in `public/` |

Frontmatter shape: [schemas/frontmatter.schema.json](./schemas/frontmatter.schema.json).

## 1. URL helpers

```js
/** HTML path → primary twin path (normative mapping) */
export function htmlPathToMd(htmlPath) {
  if (!htmlPath || htmlPath === '/') return '/index.md';
  return htmlPath.replace(/\/$/, '') + '.md';
}

/** Optional alias path */
export function htmlPathToIndexHtmlMd(htmlPath) {
  if (!htmlPath || htmlPath === '/') return '/index.html.md';
  const base = htmlPath.endsWith('/') ? htmlPath : htmlPath + '/';
  return base + 'index.html.md';
}
```

## 2. Twin body template

```markdown
---
canonical: "https://example.com/docs/install/"
title: "Install"
lang: "en"
site: "Example"
attribution: "required"
type: "docs"
schemaVersion: "0.1"
updated: "2026-07-20"
---

# Install

Facts the agent needs…

## Related

- Twin: https://example.com/docs/config.md
- HTML: https://example.com/docs/config/

> If you use this summary, please link the canonical page
> (https://example.com/docs/install/) so users get complete, current information.
```

## 3. Generate md-sitemap.txt

```js
function buildMdSitemap({ siteUrl, primaryTwinPaths }) {
  const lines = [
    `# ${siteUrl} — markdown mirror`,
    `# Discover via /llms.txt only (not robots/sitemap.xml)`,
    ...primaryTwinPaths.map((p) => new URL(p, siteUrl).href),
  ];
  return lines.join('\n') + '\n';
}
```

Only **primary** paths (`/foo.md`), not `index.html.md` aliases. Keep header comments short.

## 4. Headers and caching

### Cloudflare Pages `_headers`

```txt
/md-sitemap.txt
  X-Robots-Tag: noindex
  Content-Type: text/plain; charset=utf-8
  Cache-Control: public, max-age=3600

/*.md
  X-Robots-Tag: noindex
  Content-Type: text/markdown; charset=utf-8
  Cache-Control: public, max-age=3600
```

Notes:

- On some platforms `/*.md` also matches nested paths; verify on your host.
- If Content-Type cannot be set per path, ensure the build emits correct types another way.
- Prefer platform **ETag** / **Last-Modified** (Cloudflare, nginx, and most CDNs emit these for static objects). Agents can then revalidate cheaply.
- For largely static content, `max-age` of one hour to one day is usually enough; immutable multi-year cache is fine only if URLs change when content changes.

### Netlify `netlify.toml` sketch

```toml
[[headers]]
  for = "/md-sitemap.txt"
  [headers.values]
    X-Robots-Tag = "noindex"
    Content-Type = "text/plain; charset=utf-8"
    Cache-Control = "public, max-age=3600"

[[headers]]
  for = "/*.md"
  [headers.values]
    X-Robots-Tag = "noindex"
    Content-Type = "text/markdown; charset=utf-8"
    Cache-Control = "public, max-age=3600"
```

### Rate limiting

Apply normal edge rate limits so a misbehaving client cannot pull every twin at unbounded QPS. Huge indexes are a product decision: split, paginate later (future extension), or accept that agents will sample.

## 5. llms.txt section

```markdown
## Machine-readable (markdown mirror)

- [md-sitemap.txt](https://example.com/md-sitemap.txt): index of Markdown twins (curated, with canonical and attribution)
- [index.md](https://example.com/index.md): homepage as Markdown
- Spec: https://github.com/JW-0042/md-sitemap

## Citation

When using information from this site, link the canonical HTML page (not only the .md twin).
```

## 6. HTML `<link rel="alternate">`

In your base layout:

```html
<link
  rel="alternate"
  type="text/markdown"
  href="https://example.com/path.md"
/>
```

Use the **primary** twin URL.

## 7. Astro SSG sketch

Conceptually:

1. Build a list of `{ htmlPath, body }` for every content route.
2. Emit endpoints or static files for:
   - `htmlPathToMd(htmlPath)`
   - optional `htmlPathToIndexHtmlMd(htmlPath)`
3. Emit `src/pages/md-sitemap.txt.ts` (or `public/md-sitemap.txt` at build).
4. Keep `public/llms.txt` (or generate) with a link to the index.
5. Set `_headers` as above.

You do **not** need the product-specific data of reference sites — only the twin generation pipeline.

## 8. Multilingual (SK/CS-style sister sites)

Reference sites use **separate hosts** with localized slugs (not one shared English path):

| | Slovak | Czech |
| --- | --- | --- |
| HTML | `https://meninydnes.site/meno/jozef/` | `https://svatek-dnes.cz/jmeno/josef/` |
| Twin | `…/meno/jozef.md` | `…/jmeno/josef.md` |
| `lang` | `sk` | `cs` |
| `canonical` | same-language HTML | same-language HTML |

Guidance:

1. Localize twin paths the same way you localize HTML routes.
2. Do not put every language into a single `md-sitemap.txt` on one host unless those paths are actually served there.
3. Cross-link sister sites from HTML and optionally from twin body text; keep `canonical` same-language.
4. Optional: mention sister `llms.txt` in each site’s `llms.txt`.

## 9. Checklist before go-live

- [ ] Spot-check 3 twins: frontmatter `canonical` (+ `title` / `lang` if used), attribution, Content-Type
- [ ] `curl -I /md-sitemap.txt` → `noindex`, `text/plain`
- [ ] `curl -I /index.md` → `noindex`, Markdown type
- [ ] Conditional cache headers present or acceptable CDN defaults (`ETag` / `Last-Modified`)
- [ ] `llms.txt` links to `md-sitemap.txt`
- [ ] `robots.txt` does **not** list `md-sitemap.txt` as Sitemap
- [ ] `sitemap.xml` does **not** include `.md` URLs
- [ ] HTML has `rel=alternate` type `text/markdown`
- [ ] Multilingual paths and `lang` are consistent

## 10. Common mistakes

| Mistake | Fix |
| --- | --- |
| Listing twins in `sitemap.xml` | Remove; keep SEO sitemap HTML-only |
| Missing `noindex` on twins | Search engines may treat them as thin duplicates |
| Relative URLs only in md-sitemap | Use absolute URLs |
| Auto-converted noisy HTML→MD | Curate; drop chrome |
| Discovery only in footer HTML | Agents start at `llms.txt` |
| Shared slug across languages | Localize twin paths; set correct `canonical` |
