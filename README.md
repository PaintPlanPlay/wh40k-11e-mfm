# wh40k-mfm-scraper

> **Paint Plan Play's fork of [BSData/wh40k-11e-mfm](https://github.com/BSData/wh40k-11e-mfm).**
> The scraper is BSData's (MIT); this fork runs it for the
> [Paint Plan Play Dataset](https://github.com/PaintPlanPlay/dataset) and publishes each
> scrape as soon as it is made, without waiting for a review: the scheduled job runs four
> times a day and commits every change straight to `main`. The review happens downstream,
> when a Dataset release is built from this data. Parser fixes from upstream are taken by
> hand when this fork needs them; upstream's `data/` is never merged.

Scrapes the official Warhammer 40,000 **Munitorum Field Manual**
(<https://mfm.warhammer-community.com/en>) into clean, versioned YAML and tracks points
changes over time.

- **Dataset** — one human-readable file per faction in [`data/`](data/).
- **Change history** — deterministic YAML means `git log -p data/` is a precise record of
  every points change; the scheduled job commits it with a generated changelog whenever GW
  updates the manual.

Most data is server-rendered, so the core scrape is a plain HTTP GET + HTML parse.
**Legends** units and the expandable **"Welcome…" notes** and **"Muster Armies" rules** are
client-only, so those are captured with a headless browser (Playwright); pass `--no-legends`
to skip it.

## Quick start
```bash
pnpm install
pnpm exec playwright install chromium   # once, for Legends/notes
pnpm test                      # offline tests against saved HTML fixtures
pnpm scrape --faction necrons  # live run for one faction → data/necrons.yaml
pnpm scrape --no-legends       # fast HTTP-only run (no browser)
pnpm scrape                    # full run, all factions (with Legends + notes)
```

## How it stays correct
- The scraped shape is validated against zod schemas (`src/model.ts`) before writing —
  drift fails loudly instead of producing bad data.
- The parser is covered by tests over real saved pages (`test/fixtures/`), so site changes
  are caught in CI.
- A GitHub Action re-scrapes four times a day, commits on change, and files an issue on
  Paint Plan Play's tracker if a parse breaks.
- This fork does not announce on Discord: Paint Plan Play is told by an issue, and the
  Dataset rebuild is triggered (secret `BOT_TOKEN`, see the workflow).

## Tech
TypeScript (Node 22, ESM) · cheerio · zod · `yaml` · Vitest · Biome · pnpm.

## For maintainers / agents
See [`AGENTS.md`](AGENTS.md) for the project map and commands, [`specs/`](specs/) for the
source-of-truth contracts, and [`.agents/playbooks/`](.agents/playbooks/) for runbooks
(notably what to do when a scrape breaks).

## Disclaimer
Unofficial. Warhammer 40,000 and the Munitorum Field Manual are © Games Workshop. This
project reformats publicly available points values for personal/community reference and is
not affiliated with or endorsed by Games Workshop.
