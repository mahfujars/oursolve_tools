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

Blog posts are written in the Django admin (`oursolve_dashboard`) and served via API. The static blog pages fetch from the API.

### Files
| File | Purpose |
|------|---------|
| `blog/index.html` | Post listing, fetches `GET /dashboard/api/posts/` |
| `blog/post/index.html` | Single post view, fetches `GET /dashboard/api/posts/<slug>/` |
| `blog/.htaccess` | Clean URL routing |

### Clean URL Routing

`blog/.htaccess` rewrites `/blog/<slug>/` → `post/index.html`:
```apache
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^[a-zA-Z0-9_-]+/?$ post/index.html [L]
```

`blog/post/index.html` extracts the slug from the pathname (not just `?slug=` param):
```javascript
const params = new URLSearchParams(window.location.search);
const pathSlug = window.location.pathname.replace(/\/$/, '').split('/').pop();
const slug = params.get('slug') || (pathSlug && pathSlug !== 'post' ? pathSlug : null);
if (!slug) { window.location.href = '/blog/'; return; }
```

Post cards in `blog/index.html` link to clean URLs: `/blog/${post.slug}/`
Post cards in `index.html` (homepage preview) also use clean URL format.

### API Integration
- Fetch from `https://oursolve.com/dashboard/api/posts/` (paginated, 10/page)
- Fetch single post: `https://oursolve.com/dashboard/api/posts/<slug>/`
- API returns: `title`, `slug`, `content` (HTML), `excerpt`, `featured_image_url`, `category`, `tags`, `author`, `published_at`, `meta_title`, `meta_description`

---

## Deploy

`.cpanel.yml` copies all files to `public_html/`:
```yaml
deployment:
  tasks:
    - /bin/cp -r * /home/oursolve/public_html/
```

Push to GitHub → cPanel Git Version Control → Deploy HEAD Commit.

---

## Adding a New Tool

1. Create `tools/<tool-slug>/index.html`
2. Use the standard page structure: nav → hero (with `.tool-badge`) → main (`transform: translateY(-32px)`) → footer
3. Add tool card to `index.html` homepage grid
4. Add tool card to `tools/index.html` tools listing
5. Apply Safari copy fix pattern if the tool has copy buttons
6. Update `sitemap.xml` with the new tool URL

---

## File Structure

```
oursolve_tools/
├── index.html                       # Homepage — all tools grid + blog preview
├── tools/
│   ├── index.html                   # All tools listing page
│   ├── qr-generator/index.html
│   ├── password-generator/index.html
│   ├── word-counter/index.html
│   ├── json-formatter/index.html
│   ├── base64/index.html
│   ├── url-encoder/index.html
│   ├── hash-generator/index.html
│   ├── markdown-to-html/index.html
│   └── regex-tester/index.html
├── blog/
│   ├── index.html                   # Blog listing
│   ├── post/index.html              # Single post view
│   └── .htaccess                    # Clean URL routing
├── sitemap.xml
├── robots.txt
├── .cpanel.yml
├── CLAUDE.md                        # This file
└── README.md
```
