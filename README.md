# AI Design Checker

A tool to score Show HN projects (or any website) for common AI design patterns.
Basis for the [blog post](https://www.adriankrebs.ch/blog/design-slop/) and the HN discussion ["Scoring Show HN submissions for AI design patterns"](https://news.ycombinator.com/item?id=47864393).

Many Show HN projects have a generic sterile feeling that tells you they're AI-generated.

The scorer loads each site in a headless browser, analyzes the DOM, and reports which of 16 deterministic AI design patterns are found.
Manual verification across ~150 labeled sites suggests ~5–10% false positives.

![Browseable results report — tier filters, pattern frequency, and a card grid of every analyzed site](docs/report.png)

## Quick start

Requires Node 18+ and ~1 GB free for the bundled Chromium.

```bash
npm install
npx playwright install chromium

# Try one URL — takes ~10 s
node src/run.js --url=https://example.com

# Or run the full bundled list (urls.txt, ~900 Show HN posts)
npm run analyze

# Faster: 4 URLs in parallel — ~25 min vs ~100 min sequential, ~600 MB RAM
node src/run.js --concurrency=4
```

Results go to `results/`:
- `results/raw/<slug>.json` — per-URL signals + score
- `results/screenshots/<slug>.png` — full-page screenshot
- `results/all-results.json` — all scored entries combined

Pass `--skip-existing` to skip URLs that already have a cached result.

## Patterns

| # | Pattern | Tell |
|---|---|---|
| 1 | Templated fonts | Space Grotesk, Instrument Serif, Geist, Syne — fine in isolation, default-stack tell |
| 2 | Centered hero | H1 centered + Inter as primary font |
| 3 | VibeCode purple | Filled violet/indigo CTAs |
| 4 | Perma dark | Dark template, or muted grey body text on dark |
| 5 | All-caps headings | ≥ 3 uppercase H-tags |
| 6 | Gradients | linear/radial/conic-gradient + `background-clip: text` |
| 7 | Colored glow | Saturated `box-shadow` blur ≥ 15 px |
| 8 | Eyebrow pill | Small badge above the H1 |
| 9 | Accent stripe | Colored left/top stripe on a card with a heading |
| 10 | Icon-card grid | 3–12 cards: icon-in-badge + title + blurb |
| 11 | Numbered steps | "1·2·3" digits in styled badges |
| 12 | Stat banner | "10K+ users · 99.9% uptime · 4.9★" |
| 13 | FAQ accordion | "Frequently asked questions" with ≥ 3 Q/A items |
| 14 | Emoji nav | ≥ 40 % of nav links prefixed with an emoji |
| 15 | shadcn signature | CSS vars (`--background`), `data-slot`, Lucide, Radix attrs |
| 16 | Glassmorphism | `backdrop-filter: blur(...)` on a translucent panel |

The full rule for each pattern lives in `src/patterns/<id>.js`.

## Score

```
score = round(100 × patternsFlagged / patternsTotal)
tier  = ≥5 Heavy · 2–4 Mild · 0–1 Clean
```

## Tools

```bash
npm run scan      # → http://localhost:7788  paste a URL, see verdict
npm run label     # → http://localhost:7777  label sites for ground truth
npm run eval      # precision / recall vs dataset/labels.jsonl (165 labels shipped)
npm run report    # generate results/index.html — browsable tier-filtered grid
npm run fetch     # pull the latest 100 Show HN URLs into urls.txt
```

The scan UI launches a real browser per URL, so the first scan after starting the server is slower. The label UI lets you mark each pattern `present` / `not_present` / `skip`; saves are appended to `dataset/labels.jsonl`. The eval script compares the detector's verdict against those labels.

## Adding a pattern

Drop a file in `src/patterns/<id>.js` and append it to the list in `src/patterns/index.js`. Each pattern exports `{ id, label, shortLabel, description, category, thresholds, extract(ctx), score(signal, T) }`. `extract` is serialized via `Function.prototype.toString()` and runs in the browser — keep it self-contained (only reference `ctx.*`, no closures, no imports). `score` runs in Node and returns `{ triggered: true/false, evidence: ... }`.

## License

MIT.
