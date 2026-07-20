# Markdown Sitemap & Page Twin Protocol

**Short name:** md-sitemap  
**Version:** 0.1.0  
**Status:** Draft  
**License:** Apache-2.0  
**Compatible with:** [llms.txt](https://llmstxt.org/)

This document is **normative**. Keywords **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

---

## 1. Abstract

md-sitemap defines:

1. **Page twins** — curated Markdown documents that mirror the facts of HTML content pages for machine readers (AI agents, LLM crawlers).
2. **`md-sitemap.txt`** — a plain-text index of primary twin URLs, discovered through `llms.txt`, not through classic SEO sitemap channels.

The goal is token-efficient ingestion, stable citation of human pages, and a clean separation between **search-engine surfaces** and **agent surfaces**.

---

## 2. Terminology

| Term | Definition |
|------|------------|
| **HTML page** | A user-facing resource, typically `text/html`, intended for browsers and search engines. |
| **Twin** | A curated Markdown document that summarizes or restates the content of one HTML page for machines. |
| **Primary twin URL** | The preferred twin URL listed in `md-sitemap.txt` (see §4). |
| **md-sitemap** | The file at `/md-sitemap.txt` (or an absolute URL path ending in `md-sitemap.txt`) listing primary twin URLs. |
| **Discovery surface** | How agents learn that twins exist — primarily `/llms.txt`. |
| **Publisher** | The site operator implementing this protocol. |
| **Agent** | An automated client (LLM tool, crawler, assistant) consuming twins. |

---

## 3. Page twins

### 3.1 Existence

Publishers **SHOULD** provide a twin for each **content** HTML page they want agents to understand well (articles, product facts, day pages, docs).

Twins **MUST NOT** be a raw dump of full HTML. They **SHOULD** be curated, factual Markdown suitable for direct model context.

### 3.2 URL mapping (normative)

| HTML resource | Primary twin URL | Optional alias |
|---------------|------------------|----------------|
| Site root `/` | `/index.md` | `/index.html.md` |
| Path `/foo` or `/foo/` | `/foo.md` | `/foo/index.html.md` |
| Nested `/a/b/` | `/a/b.md` | `/a/b/index.html.md` |

Rules:

1. The **primary twin** is the bare `*.md` sibling of the HTML path (except the homepage, which is `/index.md`).
2. If an optional alias is published, it **MUST** return the **same body** as the primary twin (or redirect to it with `301`/`302`/`308`).
3. Twin paths **SHOULD** use ASCII, lowercase, hyphenated segments consistent with the site’s public URLs.
4. Query strings and fragments **MUST NOT** be required to fetch a twin.

### 3.3 HTTP response

When serving a twin, the publisher:

1. **MUST** use `Content-Type: text/markdown; charset=utf-8` (or `text/plain; charset=utf-8` if markdown type is unavailable — markdown is preferred).
2. **MUST** send `X-Robots-Tag: noindex` (and **MAY** add `nofollow` if desired for the twin itself).
3. **SHOULD** allow caching with a reasonable `Cache-Control` (e.g. hours to days for static sites).
4. **MUST NOT** require authentication for public content twins.

### 3.4 Twin document structure

A twin **SHOULD** begin with YAML frontmatter delimited by `---`.

#### Required frontmatter

| Key | Type | Description |
|-----|------|-------------|
| `canonical` | string (absolute URL) | The HTML page this twin represents. Agents **MUST** treat this as the preferred citation target for end users. |

#### Recommended frontmatter

| Key | Type | Description |
|-----|------|-------------|
| `lang` | BCP 47 tag | Document language (e.g. `sk`, `cs`, `en`). |
| `site` | string | Human site name. |
| `type` | string | Publisher-defined page class (`home`, `article`, `product`, …). |
| `attribution` | string | e.g. `required` — signals citation expectation. |
| `source` | string | Dataset or editorial source identifier. |
| `updated` | string (ISO 8601 date) | Last editorial update of the twin. |

Unknown keys **MAY** be present; agents **SHOULD** ignore keys they do not understand.

#### Body

1. **SHOULD** start with a single `#` heading matching the page topic.
2. **SHOULD** contain the facts a user would need, without site chrome (nav, cookie banners, footers).
3. **SHOULD** include an **attribution blockquote** that asks agents to link the `canonical` HTML URL when using the summary, for example:

```markdown
> If you use this summary, please link the canonical page
> (https://example.com/path/) so users get complete, current information.
```

4. **MAY** link related twin URLs (`.md`) and HTML URLs; when both exist, prefer clarity (label HTML vs twin).

### 3.5 HTML discovery of twins

HTML pages **SHOULD** include:

```html
<link rel="alternate" type="text/markdown" href="https://example.com/path.md" />
```

where `href` is the **primary** twin URL (absolute or root-relative).

---

## 4. md-sitemap.txt

### 4.1 Location

Publishers implementing **Full** conformance (see §7) **MUST** publish a file at:

```
https://{host}/md-sitemap.txt
```

Alternative paths **MAY** be used only if clearly linked from `llms.txt` with a URL whose path contains `md-sitemap` (e.g. `/ai/md-sitemap.txt`). The root path is preferred.

### 4.2 Media type and robots

1. **MUST** respond with `Content-Type: text/plain; charset=utf-8`.
2. **MUST** send `X-Robots-Tag: noindex`.
3. **MUST NOT** be listed as a `Sitemap:` in `robots.txt`.
4. **MUST NOT** be included as a URL entry in `sitemap.xml` / sitemap index intended for general web search.

### 4.3 Format

- Encoding: UTF-8.
- One **absolute** primary twin URL per non-comment line.
- Lines starting with `#` are comments and **MUST** be ignored by agents.
- Empty lines **MUST** be ignored.
- Only **primary** twin URLs **SHOULD** be listed (not optional `index.html.md` aliases).
- Order is undefined; publishers **MAY** group by section with comment headers.
- The file **MAY** begin with comments describing site name, policy, and that discovery is via `llms.txt`.

#### Example

```text
# Example Site — markdown mirror (AI/GEO surface)
# canonical HTML site: https://example.com
# linked only from /llms.txt — not from robots.txt or sitemap.xml
https://example.com/index.md
https://example.com/about.md
https://example.com/docs/getting-started.md
```

### 4.4 Completeness

`md-sitemap.txt` **SHOULD** list every primary twin the publisher wants agents to enumerate. It **MAY** omit twins that are intentionally private or experimental.

---

## 5. Discovery via llms.txt

### 5.1 Linking

Publishers **MUST** make `md-sitemap.txt` discoverable by linking it from `/llms.txt` (and **MAY** also link it from `/llms-full.txt` or similar agent entry documents).

Example fragment for `llms.txt`:

```markdown
## Machine-readable (markdown mirror)

- [md-sitemap.txt](https://example.com/md-sitemap.txt): index of Markdown twins
- [index.md](https://example.com/index.md): homepage as Markdown
```

### 5.2 Non-discovery channels

Publishers **MUST NOT** rely on `robots.txt` or SEO `sitemap.xml` as the primary discovery path for `md-sitemap.txt`. Agents **SHOULD** still try `/llms.txt` first.

---

## 6. Agent behavior (normative for consumers)

Agents that claim md-sitemap support:

1. **SHOULD** fetch `/llms.txt` when exploring a site for agent content.
2. **SHOULD** detect links to `md-sitemap.txt` (or paths containing `md-sitemap`).
3. **MAY** fetch all or a subset of twin URLs from the index according to their task budget.
4. When presenting answers to end users, agents **SHOULD** cite the HTML URL from `canonical` rather than only the twin URL.
5. Agents **MUST NOT** treat twin `noindex` as a prohibition on reading; `noindex` means “do not rank as a separate web search result,” not “do not use for agents.”
6. Agents **SHOULD** respect ordinary crawl politeness (rate limits, `Cache-Control`).

Full discovery algorithm: [AGENTS.md](./AGENTS.md).

---

## 7. Conformance levels

### 7.1 Core

A site is **Core**-conformant if:

1. Content pages of interest have twins (§3).
2. Twins use the URL mapping in §3.2 (at least primary URLs).
3. Twins are served with `noindex` (§3.3).
4. Twins include required `canonical` frontmatter (§3.4).

### 7.2 Full

A site is **Full**-conformant if it meets Core and:

1. Publishes `/md-sitemap.txt` per §4.
2. Links it from `/llms.txt` per §5.
3. Exposes `rel=alternate` type `text/markdown` on corresponding HTML pages (§3.5).

---

## 8. Security and privacy considerations

1. Twins **MUST NOT** include secrets, private PII, or non-public admin content.
2. Publishers **SHOULD** keep twin facts consistent with public HTML (avoid agent-only misleading content).
3. Agents **SHOULD** treat twin content as untrusted input (prompt injection hygiene).
4. Cross-origin twin URLs in `md-sitemap.txt` **SHOULD NOT** appear; the index **SHOULD** list same-site (or clearly related host) URLs only.

---

## 9. Versioning and extensions

1. This specification uses semantic versions. Breaking changes require a major version bump.
2. Future optional extensions (non-normative for 0.1):
   - JSON index (`md-sitemap.json`)
   - Lastmod columns
   - Multilingual hreflang for twins
3. Extensions **MUST NOT** break Core/Full parsing of the plain-text one-URL-per-line format.

---

## 10. Reference implementations

Production sites known to implement this draft are listed in [sites.md](./sites.md). Examples for copy-paste are under [examples/](./examples/).

---

## 11. Relationship to other work

See [COMPARISON.md](./COMPARISON.md). md-sitemap **complements** [llms.txt](https://llmstxt.org/); it does not supersede it.
