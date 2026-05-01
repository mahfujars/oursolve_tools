# Oursolve Tools

Free, fast, and private tools for everyday tasks — hosted at [oursolve.com](https://oursolve.com).

## Tools

| Tool | Description |
|------|-------------|
| **QR Code Generator** | Convert URLs or text into scannable QR codes, download as PNG |
| **Password Generator** | Generate strong random passwords with custom options and strength meter |
| **Word Counter** | Count words, characters, sentences, paragraphs — with reading/speaking time |

## Tech Stack

- **Pure HTML, CSS, JavaScript** — no frameworks, no build tools, no npm
- **100% client-side** — everything runs in the browser, nothing is sent to a server
- **cPanel shared hosting** — deployed via Git Version Control

## Project Structure

```
oursolve_tools/
├── index.html                        # Hub homepage
├── tools/
│   ├── qr-generator/index.html
│   ├── password-generator/index.html
│   └── word-counter/index.html
├── .cpanel.yml                       # Deployment config
├── CLAUDE.md                         # AI assistant context
└── README.md
```

## Deployment

This project deploys to cPanel using Git Version Control.

1. Add this repo as a remote in cPanel → Git Version Control
2. Push to `main`
3. cPanel runs `.cpanel.yml` which copies files to `/home/oursolve/public_html/`

## Development

No build step needed. Open any `.html` file directly in a browser to develop locally.

```bash
# Clone the repo
git clone <repo-url>
cd oursolve_tools

# Open in browser (or use VS Code Live Server)
open index.html
```

## Adding a New Tool

1. Create `tools/<tool-name>/index.html`
2. Follow the design system in `CLAUDE.md`
3. Add a card to the homepage `index.html`
4. Commit and push — cPanel auto-deploys

## License

MIT
