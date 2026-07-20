# Markdown Sitemap & Page Twin Protocol

**Short name:** md-sitemap  
**Version:** 0.1.1  
**Status:** Draft  
**License:** Apache-2.0  
**Compatible with:** [llms.txt](https://llmstxt.org/)  
**Frontmatter schema:** [schemas/frontmatter.schema.json](./schemas/frontmatter.schema.json)

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
| --- | --- |
| **HTML page** | A user-facing resource, typically `text/html`, intended for browsers and search engines. |
| **Twin** | A curated Markdown document that summarizes or restates the content of one HTML page for machines. |
| **Primary twin URL** | The preferred twin URL listed in `md-sitemap.txt` (see §4). Mapping for primary twins is normative in §3.2. |
| **md-sitemap** | The file at `/md-sitemap.txt` (or an absolute URL path ending in `md-sitemap.txt`) listing primary twin URLs. |
| **Discovery surface** | How agents learn that twins exist — primarily `/llms.txt`. |
| **Publisher** | The site operator implementing this protocol. |
| **Agent** | An automated client (LLM tool, crawler, assistant) consuming twins. |

---

## 3. Page twins

### 3.1 Existence

Publishers **SHOULD** provide a twin for each **content** HTML page they want agents to understand well (articles, product facts, day pages, docs).

Twins **MUST NOT** be a raw dump of full HTML. They **SHOULD** be curated, factual Markdown suitable for direct model context.

### 3.2 URL mapping (normative for primary twins)

The following mapping is **normative for primary twin URLs**. Publishers claiming Core or Full conformance **MUST** expose at least the primary twin at the path shown. Optional aliases are non-normative convenience URLs.

| HTML resource | Primary twin URL (normative) | Optional alias (non-normative) |
| --- | --- | --- |
| Site root `/` | `/index.md` | `/index.html.md` |
| Path `/foo` or `/foo/` | `/foo.md` | `/foo/index.html.md` |
| Nested `/a/b/` | `/a/b.md` | `/a/b/index.html.md` |

Rules:

1. The **primary twin** **MUST** be the bare `*.md` sibling of the HTML path (except the homepage, which **MUST** be `/index.md`).
2. If an optional alias is published, it **MUST** return the **same body** as the primary twin (or redirect to it with `301`, `302`, or `308`).
3. Twin paths **SHOULD** use ASCII, lowercase, hyphenated segments consistent with the site’s public URLs.
4. Query strings and fragments **MUST NOT** be required to fetch a twin.
5. Only **primary** twin URLs **SHOULD** appear in `md-sitemap.txt` (§4.3).

### 3.3 HTTP response

When serving a twin, the publisher:

1. **MUST** use `Content-Type: text/markdown; charset=utf-8` (or `text/plain; charset=utf-8` if a Markdown type is unavailable — Markdown is preferred).
2. **MUST** send `X-Robots-Tag: noindex` (and **MAY** add `nofollow` if desired for the twin itself).
3. **SHOULD** allow caching with a reasonable `Cache-Control` (for example, hours to days for static sites).
4. **SHOULD** support conditional requests via `ETag` and/or `Last-Modified` when the platform allows.
5. **MUST NOT** require authentication for public content twins.

### 3.4 Twin document structure

A twin **SHOULD** begin with YAML frontmatter delimited by `---`.

Machine-readable schema: [schemas/frontmatter.schema.json](./schemas/frontmatter.schema.json).

#### Required frontmatter

| Key | Type | Description |
| --- | --- | --- |
| `canonical` | string (absolute URL) | The HTML page this twin represents. Agents **MUST** treat this as the preferred citation target for end users. |

#### Recommended frontmatter

| Key | Type | Description |
| --- | --- | --- |
| `title` | string | Short human title for the page (agents **SHOULD** prefer this over guessing from the first heading). |
| `lang` | BCP 47 tag | Document language (for example, `sk`, `cs`, `en`). |
| `site` | string | Human site name. |
| `type` | string | Publisher-defined page class (`home`, `article`, `product`, …). |
| `attribution` | string | For example, `required` — signals citation expectation. |
| `source` | string | Dataset or editorial source identifier. |
| `updated` | string (ISO 8601 date) | Last editorial update of the twin. |
| `schemaVersion` | string | Spec version this twin was written against (for example, `"0.1"`). Recommended for future-proofing. |

Unknown keys **MAY** be present; agents **SHOULD** ignore keys they do not understand.

#### Body

1. **SHOULD** start with a single `#` heading matching the page topic (align with `title` when present).
2. **SHOULD** contain the facts a user would need, without site chrome (nav, cookie banners, footers).
3. **SHOULD** include an **attribution blockquote** that asks agents to link the `canonical` HTML URL when using the summary, for example:

```markdown
> If you use this summary, please link the canonical page
> (https://example.com/path/) so users get complete, current information.
```

4. **MAY** link related twin URLs (`.md`) and HTML URLs; when both exist, label them clearly (HTML vs twin).

### 3.5 HTML discovery of twins

HTML pages **SHOULD** include:

```html
<link rel="alternate" type="text/markdown" href="https://example.com/path.md" />
```

where `href` is the **primary** twin URL (absolute or root-relative).

### 3.6 Multilingual sites

Publishers with multiple languages **SHOULD**:

1. Set `lang` on each twin to the language of that twin’s body.
2. Prefer **localized primary twin paths** that match localized HTML slugs (for example, SK `/meno/jozef.md` and CS `/jmeno/josef.md` on sister hosts), rather than forcing one language’s path everywhere.
3. Put `canonical` on the **same-language HTML** page users should open.
4. **MAY** add optional frontmatter such as `hreflang` maps or sibling URLs in the body for cross-language navigation; agents **SHOULD** treat those as hints, not as a replacement for `canonical`.

A twin **SHOULD NOT** mix languages in a way that obscures which HTML page is the citation target.

---

## 4. md-sitemap.txt

### 4.1 Location

Publishers implementing **Full** conformance (see §7) **MUST** publish a file at:

```
https://{host}/md-sitemap.txt
```

Alternative paths **MAY** be used only if clearly linked from `llms.txt` with a URL whose path contains `md-sitemap` (for example, `/ai/md-sitemap.txt`). The root path is preferred.

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
- Order is undefined; publishers **MAY** group by section with short comment headers.
- The file **MAY** begin with a few comment lines (site name, discovery via `llms.txt`). Comments **SHOULD** stay brief.

#### Example

```text
# Example Site — markdown mirror
# Discover via /llms.txt only (not robots/sitemap.xml)
https://example.com/index.md
https://example.com/about.md
https://example.com/docs/getting-started.md
```

### 4.4 Completeness and size

`md-sitemap.txt` **SHOULD** list every primary twin the publisher wants agents to enumerate. It **MAY** omit twins that are intentionally private or experimental.

Publishers **SHOULD** keep indexes practical for agent budgets. Extremely large lists without prioritization may be truncated by clients (see [AGENTS.md](./AGENTS.md)).

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
6. Agents **SHOULD** respect ordinary crawl politeness (rate limits, `Cache-Control`, conditional requests).
7. Agents **SHOULD** degrade gracefully when twins are missing, incomplete, or inconsistent (for example, fall back to HTML or other sources). See [AGENTS.md](./AGENTS.md).

Full discovery algorithm: [AGENTS.md](./AGENTS.md).

---

## 7. Conformance levels

### 7.1 Core

A site is **Core**-conformant if:

1. Content pages of interest have twins (§3).
2. Twins use the **primary** URL mapping in §3.2.
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
3. Agents **SHOULD** treat twin content as untrusted input (prompt-injection hygiene).
4. Cross-origin twin URLs in `md-sitemap.txt` **SHOULD NOT** appear; the index **SHOULD** list same-site (or clearly related host) URLs only.
5. Publishers **SHOULD** apply normal HTTP rate-limiting and abuse controls. A huge or hostile `md-sitemap.txt` can waste agent resources; agents **SHOULD** cap downloads (see [AGENTS.md](./AGENTS.md)).
6. Agents **SHOULD NOT** blindly follow an unbounded number of twin URLs from an untrusted host.

---

## 9. Versioning and extensions

1. This specification uses semantic versions. Breaking changes require a major version bump.
2. Twins **SHOULD** set `schemaVersion` (for example, `"0.1"`) when practical so agents can branch on format evolution.
3. Future optional extensions (non-normative for 0.1.x):
   - JSON index (`md-sitemap.json`)
   - Lastmod columns in the text index
   - Structured hreflang maps in frontmatter
4. Extensions **MUST NOT** break Core/Full parsing of the plain-text one-URL-per-line format.

---

## 10. Reference implementations

Production sites known to implement this draft are listed in [sites.md](./sites.md). Examples for copy-paste are under [examples/](./examples/).

---

## 11. Relationship to other work

See [COMPARISON.md](./COMPARISON.md). md-sitemap **complements** [llms.txt](https://llmstxt.org/); it does not supersede it.
