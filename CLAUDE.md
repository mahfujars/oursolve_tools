# oursolve_tools — Claude Context

## What Is This
Static HTML/CSS/JS site at oursolve.com. No build step, no frameworks, no npm. Tools live in `tool/<slug>/index.html`, deployed to `public_html/` via cPanel Git.

- **Live URL:** https://oursolve.com/
- **GitHub:** https://github.com/mahfujars/oursolve_tools
- **Deploy target:** `/home/oursolve/public_html/`
- **Git repo on server:** `/home/oursolve/oursolve_tools/` (not inside public_html)

---

## Hosting Constraints — CRITICAL

**No SSH or terminal access on cPanel.** All server-side operations via:
- **File Manager** — browse/edit/delete files
- **Git Version Control** — "Deploy HEAD Commit" runs `.cpanel.yml`

Never suggest terminal commands.

---

## Core Rules (never break)

- **Pure HTML + CSS + JS only.** No npm, no frameworks, no build tools.
- **100% client-side.** Every tool works without any backend.
- **No tracking, no ads, no sign-ups.** No data leaves the browser.
- **Mobile-first and responsive.** Works on phones and tablets.
- **CDN libraries OK** only when feature requires non-trivial pure-JS AND library has SRI hash.

---

## Design System

### Colors (navy/blue — NOT indigo)

| Token | Light mode | Dark mode |
|-------|-----------|-----------|
| `--primary` | `#1C4D8D` | `#4d8fd4` |
| `--primary-dark` | `#093C5D` | `#93c5fd` |
| `--primary-light` | `#dbeafe` | `#1e3a5f` |
| `--bg` | `#f8fafc` | `#0f172a` |
| `--surface` | `#ffffff` | `#1e293b` |
| `--text` | `#0f172a` | `#f1f5f9` |
| `--text-muted` | `#64748b` | `#94a3b8` |
| `--border` | `#e2e8f0` | `#334155` |
| `--hero-gradient` | `linear-gradient(135deg, #051e3e, #093C5D, #1C4D8D)` | darker variant |

All tokens defined in `tool/shared.css` as CSS vars with `[data-theme="dark"]` overrides.

### Dark/Light Mode
- Toggle button (🌙/☀️) in nav on every page
- Preference saved to `localStorage` key `theme`
- Respects `prefers-color-scheme` on first visit
- `data-theme="dark"` set on `<html>` element via inline script in `<head>` (before paint)

### Page Structure (all tool pages)
```
<nav class="site-nav"> (loaded from /tool/header.html via JS fetch)
  ↓
.hero (var(--hero-gradient), tool badge + h1 + subtitle)
  ↓
main (max-width varies, transform: translateY(-32px))
  ↓
<footer> (loaded from /tool/footer.html via JS fetch)
```

### Shared Files (tool/)
| File | Purpose |
|------|---------|
| `tool/shared.css` | All CSS vars, dark mode vars, nav, hero, footer, common element styles |
| `tool/header.html` | Nav HTML snippet — fetched and injected by every tool page |
| `tool/footer.html` | Footer HTML snippet — fetched and injected by every tool page |
| `tool/tools.json` | Tool manifest for homepage + WP tools page |

Every tool `index.html` links `shared.css` and fetches header/footer:
```html
<link rel="stylesheet" href="/tool/shared.css">
```
```js
fetch('/tool/header.html').then(r=>r.text()).then(html=>{ document.getElementById('tool-header').outerHTML=html; ... });
fetch('/tool/footer.html').then(r=>r.text()).then(html=>{ document.getElementById('tool-footer').outerHTML=html; });
```

---

## URL Structure

| URL | What serves it |
|-----|---------------|
| `/` | WP homepage (`front-page.php`) — shows 6 featured tools + blog preview |
| `/tools/` | WP page with "Tools Listing" template (`page-tools.php`) — all tools, search, filter |
| `/tool/<slug>/` | Static HTML from `public_html/tool/<slug>/index.html` |
| `/blog/` | Static `blog/index.html` — fetches WP REST API |
| `/blog/<slug>/` | Static `blog/post/index.html` — fetches WP REST API |

Note: `/tools/` is a WP page (no static file). `/tool/` is a real directory with static files.

---

## Tool Inventory

| Tool | Path | Category | Notes |
|------|------|----------|-------|
| QR Code Generator | `tool/qr-generator/` | Generator | qrcode.js from cdnjs (SRI hash) |
| Password Generator | `tool/password-generator/` | Security | `crypto.getRandomValues()`, Safari copy fix |
| Word Counter | `tool/word-counter/` | Writing | Pure JS |
| JSON Formatter | `tool/json-formatter/` | Developer | Format/minify/validate, syntax highlight |
| Base64 Encoder | `tool/base64/` | Encoding | Encode/decode tabs, file drag & drop |
| URL Encoder | `tool/url-encoder/` | Developer | encodeURIComponent + encodeURI modes |
| Hash Generator | `tool/hash-generator/` | Security | SHA-1/256/384/512 via Web Crypto API |
| Markdown to HTML | `tool/markdown-to-html/` | Writing | Live split preview, pure-JS parser |
| Regex Tester | `tool/regex-tester/` | Developer | Real-time highlight, capture groups, all flags |
| Muslim Baby Names | `tool/muslim-names/` | Islamic | 1000 names, `p:1` = priority (99 names derivatives) |

---

## tool/tools.json — Dynamic Manifest

`front-page.php` (WP homepage) and `page-tools.php` (WP tools page) both fetch `/tool/tools.json` to render tool cards. No PHP edit needed to add a new tool.

Entry format:
```json
{"slug":"tool-slug","name":"Tool Name","category":"Category","icon":"emoji","iconClass":"icon-css-class","desc":"Short description."}
```

Categories: `Generator`, `Security`, `Writing`, `Developer`, `Encoding`, `Islamic`

Icon CSS classes defined in `tool/shared.css`: `icon-qr`, `icon-pass`, `icon-word`, `icon-json`, `icon-b64`, `icon-url`, `icon-hash`, `icon-md`, `icon-rx`, `icon-islamic`

---

## Copy Button Pattern — Safari Fix

**Problem:** Mac Safari clears `window.event` after `await`, so `event.target` is null in async copy handlers.

**Fix:** Always pass `this` from the `onclick` attribute. Never use `event.target` in async functions.

```html
<button onclick="copyField('output-id', this)">Copy</button>
```
```javascript
async function copyField(id, btn) {
  const val = document.getElementById(id).value;
  if (!val) return;
  const ok = await clip(val);
  if (ok) { const o = btn.textContent; btn.textContent = '✅ Copied!'; setTimeout(() => btn.textContent = o, 1500); }
}
```

---

## Blog System

Blog posts written in **WordPress Admin** (`oursolve.com/wp-admin/`). Static blog pages fetch from WP REST API.

### Files
| File | Purpose |
|------|---------|
| `blog/index.html` | Post listing — fetches WP REST API |
| `blog/post/index.html` | Single post view — fetches WP REST API |
| `blog/.htaccess` | Clean URL: `/blog/<slug>/` → `post/index.html` |

### WP REST API
- Listing: `GET /wp-json/wp/v2/posts?per_page=10&page=1&_embed=1`
- Single: `GET /wp-json/wp/v2/posts?slug=<slug>&_embed=1`
- Pagination: `X-WP-TotalPages` response header
- `_embed=1` includes featured image, categories/tags, author

---

## Deploy

`.cpanel.yml` copies only static files — never touches WP core:
```yaml
deployment:
  tasks:
    - /bin/cp index.html /home/oursolve/public_html/index.html
    - /bin/cp sitemap.xml /home/oursolve/public_html/sitemap.xml
    - /bin/cp robots.txt /home/oursolve/public_html/robots.txt
    - /bin/cp -R tool /home/oursolve/public_html/
    - /bin/cp -R blog /home/oursolve/public_html/
```

Push to GitHub → cPanel Git → **Update from Remote** → **Deploy HEAD Commit**.

---

## Adding a New Tool

1. Create `tool/<slug>/index.html`
2. Link `shared.css`, add `<div id="tool-header"></div>` and `<div id="tool-footer"></div>`, add header/footer fetch JS + dark mode init script
3. Add entry to `tool/tools.json` — homepage and tools page auto-update
4. Apply Safari copy fix if tool has copy buttons
5. Update `sitemap.xml`

---

## File Structure

```
oursolve_tools/
├── index.html                       # Homepage (WP handles /, this file unused by WP)
├── tool/
│   ├── shared.css                   # Shared CSS vars, dark mode, nav, hero, footer styles
│   ├── header.html                  # Shared nav snippet (fetched by each tool)
│   ├── footer.html                  # Shared footer snippet (fetched by each tool)
│   ├── tools.json                   # Tool manifest — add entry here to add new tool
│   ├── qr-generator/index.html
│   ├── password-generator/index.html
│   ├── word-counter/index.html
│   ├── json-formatter/index.html
│   ├── base64/index.html
│   ├── url-encoder/index.html
│   ├── hash-generator/index.html
│   ├── markdown-to-html/index.html
│   ├── regex-tester/index.html
│   └── muslim-names/
│       ├── index.html
│       └── names.json               # 1000 names, p:1 = Allah's 99 names derivatives (shown first)
├── blog/
│   ├── index.html                   # Blog listing — fetches WP REST API
│   ├── post/index.html              # Single post — fetches WP REST API
│   └── .htaccess                    # Clean URL routing
├── sitemap.xml
├── robots.txt
├── .cpanel.yml
├── CLAUDE.md                        # This file
└── README.md
```
