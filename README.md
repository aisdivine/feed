# The Feed — a personal news dashboard

A single static page that aggregates Hacker News, Reddit, world news, and your
own Twitter/X list, then uses Claude to summarize, tag, and relevance-rank every
item against *your* interests. Rebuilt on a schedule by GitHub Actions and
served free on GitHub Pages. No servers, no database.

```
GitHub Actions (cron, every 2h)
  └─ python build.py
       ├─ fetch RSS / Reddit          (sources.yml)
       ├─ dedup
       ├─ Claude: summarize + tag + score   (optional — needs API key)
       ├─ rank by relevance × recency × source weight
       └─ render index.html  →  commit  →  GitHub Pages serves it
```

## Setup (≈10 minutes)

1. **Create a repo** and drop these files in it. Push to `main`.

2. **Enable Pages:** repo *Settings → Pages → Build and deployment → Source:
   "Deploy from a branch" → `main` / root*. Your site lands at
   `https://<you>.github.io/<repo>/`.

3. **Add your Anthropic key** (for the summarization layer): *Settings →
   Secrets and variables → Actions → New repository secret* named
   `ANTHROPIC_API_KEY`. Get a key at <https://console.anthropic.com>.
   *(Skip this and the build still runs — you just get raw titles, no summaries
   or relevance filtering.)*

4. **Run it:** *Actions tab → "Build feed" → Run workflow*. It builds
   `index.html`, commits it, and Pages publishes within a minute.

5. **Wire up Twitter/X** (optional but recommended): the feed is only as good as
   its sources. See the commented block at the bottom of `sources.yml` — make a
   private Twitter List, run [RSSHub](https://docs.rsshub.app)
   (`docker run -d -p 1200:1200 diygod/rsshub`), and point a source at the list
   route.

## Tuning your feed

Everything lives in **`sources.yml`** — no code changes needed:

- **`interests`** — a sentence describing what you care about. This is what
  Claude scores each item against. Rewrite it and the whole feed reshapes.
- **`min_score`** — items scoring below this (0–10) are dropped. Raise it for a
  tighter feed, lower it to see more.
- **`sources`** — add any RSS feed or subreddit. `weight` floats a source up or
  down in the ranking.

## Cost

- **GitHub Pages + Actions:** free. A 2-hour cron is ~360 runs/month at ~1 min
  each — well under the 2,000 free minutes.
- **Claude:** the only paid piece, and small — one batched call per ~40 items
  per build. Default model is `claude-opus-4-8` for best summaries; for a
  cheaper high-frequency cron, set a repo **variable** `FEED_MODEL` to
  `claude-haiku-4-5` or `claude-sonnet-4-6`.

## Run locally

```bash
pip install -r requirements.txt
export ANTHROPIC_API_KEY=sk-ant-...   # optional
python build.py
open index.html
```
