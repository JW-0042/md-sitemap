# Implementation guide

This document is **non-normative**. Normative rules live in [SPEC.md](./SPEC.md).

## Architecture choices

| Approach | When to use |
|----------|-------------|
| **Static files at build time** | SSG sites (Astro, Next export, Hugo, Eleventy) — preferred |
| **Edge/server routes** | Dynamic pages where twins are generated on demand |
| **Hybrid** | SSG for most pages + dynamic for a few |

Production pattern used by reference sites: **generate all twins at build**, deploy as static assets next to HTML, set headers in platform config (`_headers`, `vercel.json`, nginx, etc.).

## 1. URL helpers

```js
/** HTML path → primary twin path */
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
lang: "en"
site: "Example"
attribution: "required"
type: "docs"
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
    `# ${siteUrl} — markdown mirror (AI/GEO surface)`,
    `# linked only from /llms.txt — not from robots.txt or sitemap.xml`,
    ...primaryTwinPaths.map((p) => new URL(p, siteUrl).href),
  ];
  return lines.join('\n') + '\n';
}
```

Only **primary** paths (`/foo.md`), not `index.html.md` aliases.

## 4. Headers (Cloudflare Pages `_headers`)

```txt
/md-sitemap.txt
  X-Robots-Tag: noindex
  Content-Type: text/plain; charset=utf-8

/*.md
  X-Robots-Tag: noindex
  Content-Type: text/markdown; charset=utf-8
```

Notes:

- On some platforms `/*.md` also matches nested paths; verify on your host.
- If Content-Type cannot be set per path, ensure build emits correct types another way.

### Netlify `netlify.toml` sketch

```toml
[[headers]]
  for = "/md-sitemap.txt"
  [headers.values]
    X-Robots-Tag = "noindex"
    Content-Type = "text/plain; charset=utf-8"

[[headers]]
  for = "/*.md"
  [headers.values]
    X-Robots-Tag = "noindex"
    Content-Type = "text/markdown; charset=utf-8"
```

## 5. llms.txt section

```markdown
## Machine-readable (markdown mirror)

- [md-sitemap.txt](https://example.com/md-sitemap.txt): index of Markdown twins (curated, with canonical + attribution)
- [index.md](https://example.com/index.md): homepage as Markdown

## Citation

When using information from this site, link the canonical HTML page (not only the .md twin).
```

Optional: link this specification:

```markdown
- Spec: https://github.com/JW-0042/md-sitemap
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

## 8. Checklist before go-live

- [ ] Spot-check 3 twins: frontmatter `canonical`, attribution, Content-Type
- [ ] `curl -I /md-sitemap.txt` → `noindex`, `text/plain`
- [ ] `curl -I /index.md` → `noindex`, markdown type
- [ ] `llms.txt` links to `md-sitemap.txt`
- [ ] `robots.txt` does **not** list `md-sitemap.txt` as Sitemap
- [ ] `sitemap.xml` does **not** include `.md` URLs
- [ ] HTML has `rel=alternate` type `text/markdown`

## 9. Common mistakes

| Mistake | Fix |
|---------|-----|
| Listing twins in `sitemap.xml` | Remove; keep SEO sitemap HTML-only |
| Missing `noindex` on twins | Search engines may treat them as thin duplicates |
| Relative URLs only in md-sitemap | Use absolute URLs |
| Auto-converted noisy HTML→MD | Curate; drop chrome |
| Discovery only in footer HTML | Agents start at `llms.txt` |
