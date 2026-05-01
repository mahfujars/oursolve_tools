# oursolve_tools — Claude Context

## What Is This
Static HTML/CSS/JS site at oursolve.com. No build step, no frameworks, no npm. Every tool is a self-contained single HTML file deployed directly to `/home/oursolve/public_html/` via cPanel Git.

- **Live URL:** https://oursolve.com/
- **GitHub:** https://github.com/mahfujars/oursolve_tools
- **Deploy target:** `/home/oursolve/public_html/`

---

## Hosting Constraints — CRITICAL

**No SSH or terminal access on cPanel.** All server-side operations via:
- **File Manager** — browse/edit/delete files
- **Git Version Control** — "Deploy HEAD Commit" runs `.cpanel.yml`

Never suggest terminal commands. For file edits, use File Manager as fallback.

---

## Core Rules (never break)

- **Pure HTML + CSS + JS only.** No npm, no frameworks, no build tools.
- **100% client-side.** Every tool works without any backend.
- **No tracking, no ads, no sign-ups.** No data leaves the browser.
- **Mobile-first and responsive.** Works on phones and tablets.
- **CDN libraries OK** only when a feature requires non-trivial pure-JS implementation AND library has SRI hash.

---

## Design System

| Token | Value |
|-------|-------|
| Primary color | `#6366f1` (indigo) |
| Primary dark | `#4f46e5` |
| Primary light | `#e0e7ff` |
| Background | `#f8fafc` |
| Surface | `#ffffff` |
| Text | `#0f172a` |
| Text muted | `#64748b` |
| Border | `#e2e8f0` |
| Border radius | `16px` (cards), `10px` (inputs), `8px` (buttons) |
| Hero gradient | `linear-gradient(135deg, #1e1b4b, #312e81, #4338ca)` |
| Font | `-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif` |

### Page Structure (all tool pages)
```
nav (breadcrumb: Oursolve / Tool Name)
  ↓
.hero (gradient background, tool badge + h1 + subtitle)
  ↓
main (max-width varies, transform: translateY(-32px) — overlaps hero bottom)
  ↓
footer (centered, link back to all tools)
```

---

## Tool Inventory

| Tool | Path | Category | Notes |
|------|------|----------|-------|
| QR Code Generator | `tools/qr-generator/` | Utility | Uses qrcode.js from cdnjs (SRI hash) |
| Password Generator | `tools/password-generator/` | Security | `crypto.getRandomValues()`, copy Safari fix applied |
| Word Counter | `tools/word-counter/` | Writing | Pure JS |
| JSON Formatter | `tools/json-formatter/` | Developer | Format/minify/validate, syntax highlight, auto-format on paste |
| Base64 Encoder | `tools/base64/` | Developer | Encode/decode tabs, file drag & drop |
| URL Encoder | `tools/url-encoder/` | Developer | encodeURIComponent + encodeURI modes, swap buttons |
| Hash Generator | `tools/hash-generator/` | Security | SHA-1/256/384/512 via Web Crypto API, text + file input |
| Markdown to HTML | `tools/markdown-to-html/` | Writing | Live split preview, pure-JS parser, tables/code/blockquotes |
| Regex Tester | `tools/regex-tester/` | Developer | Real-time highlight, match table, capture groups, all flags |

---

## Copy Button Pattern — Safari Fix

**Problem:** Mac Safari clears `window.event` after `await`, so `event.target` is null in async copy handlers.

**Fix:** Always pass `this` from the `onclick` attribute. Never use `event.target` in async functions.

```html
<!-- CORRECT -->
<button onclick="copyField('output-id', this)">Copy</button>

<!-- WRONG — breaks on Safari -->
<button onclick="copyField('output-id')">Copy</button>
```

```javascript
async function clip(text) {
  try { await navigator.clipboard.writeText(text); return true; } catch {}
  try {
    const ta = Object.assign(document.createElement('textarea'), {value: text});
    ta.style.cssText = 'position:fixed;opacity:0';
    document.body.appendChild(ta); ta.select();
    const ok = document.execCommand('copy');
    document.body.removeChild(ta); return ok;
  } catch { return false; }
}

async function copyField(id, btn) {
  const val = document.getElementById(id).value;
  if (!val) return;
  const ok = await clip(val);
  if (ok) { const o = btn.textContent; btn.textContent = '✅ Copied!'; setTimeout(() => btn.textContent = o, 1500); }
}
```

The `execCommand('copy')` fallback handles environments where `navigator.clipboard` is unavailable (non-HTTPS, Safari quirks).

---

## Blog System

Blog posts written in **WordPress Admin** (`oursolve.com/wp-admin/`). Static blog pages fetch from WP REST API.

### Files
| File | Purpose |
|------|---------|
| `blog/index.html` | Post listing, fetches `GET /wp-json/wp/v2/posts` |
| `blog/post/index.html` | Single post view, fetches `GET /wp-json/wp/v2/posts?slug=<slug>&_embed=1` |
| `blog/.htaccess` | Clean URL routing |

### Clean URL Routing

`blog/.htaccess` rewrites `/blog/<slug>/` → `post/index.html`:
```apache
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^[a-zA-Z0-9_-]+/?$ post/index.html [L]
```

`blog/post/index.html` extracts slug from pathname:
```javascript
const pathSlug = window.location.pathname.replace(/\/$/, '').split('/').pop();
const slug = params.get('slug') || (pathSlug && pathSlug !== 'post' ? pathSlug : null);
```

### WP REST API
- Post listing: `GET /wp-json/wp/v2/posts?per_page=10&page=1&_embed=1`
- Single post: `GET /wp-json/wp/v2/posts?slug=<slug>&_embed=1`
- Categories: `GET /wp-json/wp/v2/categories?hide_empty=true`
- Pagination: `X-WP-TotalPages` response header
- `_embed=1` includes: `wp:featuredmedia` (image), `wp:term` (categories/tags), `author`

### WP Post Fields Used
| Field | Value |
|-------|-------|
| Title | `post.title.rendered` |
| Content | `post.content.rendered` |
| Excerpt | `post.excerpt.rendered` (strip HTML tags) |
| Date | `post.date` |
| URL | `post.link` |
| Featured image | `post._embedded['wp:featuredmedia'][0].source_url` |
| Categories | `post._embedded['wp:term'][0]` |
| Tags | `post._embedded['wp:term'][1]` |
| Author | `post._embedded.author[0].name` |

---

## Deploy

WordPress is installed at `public_html/` root. `.cpanel.yml` copies only static tool files — never touches WP files:
```yaml
deployment:
  tasks:
    - /bin/cp index.html /home/oursolve/public_html/index.html
    - /bin/cp sitemap.xml /home/oursolve/public_html/sitemap.xml
    - /bin/cp robots.txt /home/oursolve/public_html/robots.txt
    - /bin/cp -R tools /home/oursolve/public_html/
    - /bin/cp -R blog /home/oursolve/public_html/
```

**Git repo cloned to:** `/home/oursolve/oursolve_tools/` (not public_html)

Push to GitHub → cPanel Git Version Control → **Update from Remote** → **Deploy HEAD Commit**.

---

## tools/tools.json — Dynamic Tool Manifest

Homepage (`index.html`) and tools listing (`tools/index.html`) both fetch `tools/tools.json` via JS to render tool cards dynamically. No HTML editing needed to add a new tool to the grid.

Entry format:
```json
{"slug":"tool-slug","name":"Tool Name","category":"Category","icon":"emoji","iconClass":"icon-css-class","desc":"Short description."}
```

Categories in use: `Generator`, `Security`, `Writing`, `Developer`

---

## Adding a New Tool

1. Create `tools/<tool-slug>/index.html`
2. Use standard page structure: nav → hero (with `.tool-badge`) → main (`transform: translateY(-32px)`) → footer
3. **Add one entry to `tools/tools.json`** — homepage and tools listing auto-update
4. Apply Safari copy fix pattern if tool has copy buttons
5. Update `sitemap.xml` with new tool URL

---

## File Structure

```
oursolve_tools/
├── index.html                       # Homepage — tools grid + blog preview (fetches tools.json)
├── tools/
│   ├── index.html                   # All tools listing (fetches tools.json)
│   ├── tools.json                   # Tool manifest — add entry here to add tool to site
│   ├── qr-generator/index.html
│   ├── password-generator/index.html
│   ├── word-counter/index.html
│   ├── json-formatter/index.html
│   ├── base64/index.html
│   ├── url-encoder/index.html
│   ├── hash-generator/index.html
│   ├── markdown-to-html/index.html
│   ├── regex-tester/index.html
│   └── muslim-names/index.html      # names loaded from names.json (1000 names, p:1 = priority)
├── blog/
│   ├── index.html                   # Blog listing — fetches WP REST API
│   ├── post/index.html              # Single post view — fetches WP REST API
│   └── .htaccess                    # Clean URL routing: /blog/<slug>/ → post/index.html
├── sitemap.xml
├── robots.txt
├── .cpanel.yml
├── CLAUDE.md                        # This file
└── README.md
```
