# SmartTools

A collection of handy single-page web tools, hosted on GitHub Pages.

**Live site:** `https://<your-username>.github.io/smarttools/`

## Tools

| Tool | Description |
|------|-------------|
| [JSON Formatter](tools/json-formatter.html) | Pretty-print, minify, and validate JSON with syntax highlighting |
| [Timezone Converter](tools/timezone-converter.html) | Convert time across multiple timezones simultaneously |
| [SG Tax Calculator](tools/sg-tax-calculator.html) | Singapore personal income tax calculator (YA 2025) |

## Adding a New Tool

1. Create a new HTML file in `tools/` (copy an existing one as template)
2. Add a card entry in `index.html`
3. Push to `main` — GitHub Actions will deploy automatically

## Development

No build step required. Open any HTML file directly in a browser for local development.

## Deployment

This site is automatically deployed to GitHub Pages via GitHub Actions on every push to `main`.
