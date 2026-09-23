# Duress-Aware Wallet Security Standard — site package

Static, mobile-first project proposal you can drop onto a personal site.

## Suggested personal-site blurb

> **Duress-Aware Wallet** — A feature-layer proposal for self-custody that survives coercion: decoy vault + duress password, geofenced second factor, time-locks / multi-sig on large sends, optional app masking, and a threat-model matrix of covers / partials / gaps. Built as a readable two-part HTML reader plus a phone-friendly matrix.

## Suggested slug

`/projects/duress-aware-wallet` → serve this folder (entry: `index.html`).

## Entry URLs (relative)

| Page | Path |
|------|------|
| Landing | `index.html` |
| Part 1 | `part1.html` |
| Part 2 | `part2.html` |
| Threat matrix (HTML) | `matrix.html` |
| Matrix image (phone) | `matrix.png` |
| Matrix image (desktop wide) | `matrix-desktop.png` |
| Full proposal (HTML) | `security-standard.html` |
| Full proposal (Markdown) | `security-standard.md` |
| Findings (HTML / MD) | `findings.html` / `findings.md` |

## Publish notes

- One folder is enough: copy or rsync `site/` to your host.
- No build step. Open `index.html` locally or behind any static host.
- Do not rewrite proposal wording when embedding — link out to these pages.
- Phone CSS: Part 1 / Part 2 / matrix use swipeable tables + card-stack matrix under 640px.
