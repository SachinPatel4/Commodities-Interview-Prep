# Commodities Interview Prep

A self-hosting toolkit for preparing for commodities trading interviews. Built for a
Glencore candidate; works for any physical trading house. It gives you:

1. **A daily brief** in `briefs/` with prices, spreads, the five stories that matter, company
   news, three questions a trader might ask you, and a view of the day. Generated every
   morning by GitHub Actions in your own fork.
2. **A question bank** in `questions/` (technical, market-view, behavioural, mental maths)
   with model answers and the pushback a trader would give.
3. **A mock-interview skill** for Claude Code, plus a paste-in prompt for claude.ai chat.
4. **A networking cheat sheet** in `networking/` for the reception on 8 Sep 2026.
5. **A dashboard** (`docs/index.html`) on GitHub Pages: prices, spreads, curves, 90-day
   charts, headlines, the brief, the cheat sheet, and the mock interviewer in one page.

Everything runs on free tiers. Nothing depends on the person who built it.

Handing it to someone else? Follow [HANDOVER.md](HANDOVER.md).

## Quick start (10 minutes)

1. **Fork this repo** to your own GitHub account (keep it public: Actions minutes are free).
2. **Enable Actions** on your fork (Actions tab, "I understand my workflows, go ahead and enable them").
3. **Optional but recommended, add secrets** (Settings, Secrets and variables, Actions):
   - `CLAUDE_CODE_OAUTH_TOKEN` - turns on the analysis tier of the brief (stories, questions, view) and bills it to your Claude subscription rather than an API key. Install the Claude Code CLI (`curl -fsSL https://claude.ai/install.sh | bash`), run `claude setup-token`, and paste the printed token in as the secret. It lasts a year; needs a Pro, Max, Team or Enterprise plan. Without it you still get prices, spreads and headlines.
   - `ANTHROPIC_API_KEY` from [console.anthropic.com](https://console.anthropic.com) - the alternative to the token above, billed as API usage. Set one or the other, not both; the token wins.
   - `OILPRICEAPI_KEY` from [oilpriceapi.com](https://www.oilpriceapi.com/auth/signup) (free tier). Adds source-timestamped Brent, WTI, Henry Hub, TTF, JKM, coal and EU carbon.
   - Optional variable `BRIEF_MODEL` (Settings, Variables) to override the model, default `claude-opus-5`.
4. **Run it once by hand**: Actions, "Daily commodities brief", "Run workflow". A file appears in `briefs/` a few minutes later. From then on it runs daily at 06:30 UK time, with a retry slot at 09:00 because GitHub sometimes skips or delays scheduled runs. A manual run replaces that day's brief; scheduled runs never overwrite one.
5. **Turn on the dashboard**: Settings, Pages, Source = "GitHub Actions". Then Actions, "Dashboard (GitHub Pages)", "Run workflow". Your page is at `https://<you>.github.io/<repo>/`. It rebuilds after each daily brief and hourly on weekdays.
6. **Read the brief on your phone** from the dashboard, the GitHub app, or the repo page.

## The dashboard

`docs/index.html` is one file with no build step. The Pages workflow writes `site/data.json`
(latest prices with 90-day history, spreads, headlines, the latest brief, the cheat sheet and
the parsed question bank) next to it. Tabs: Overview, Charts, News, Brief, Interview, Cheat sheet.

The **Interview** tab runs the mock interview inside the page. It marks your answers with
Claude in one of two ways, detected automatically:

- Inside claude.ai (when the page is published as an artifact), it asks Claude through your
  own claude.ai account. No key. You are asked to allow it once.
- On GitHub Pages, open Settings in the tab and paste your own Anthropic API key. It is kept
  in that browser's local storage only and sent straight to Anthropic. Use a key you can revoke.

To publish the page as a claude.ai artifact from your own Claude Code session:
`python scripts/build_dashboard_data.py --inline docs/index.html --out site/preview.html`
and ask Claude to publish `site/preview.html` as an artifact with the `sample` capability.
That copy is a snapshot; the Pages version refreshes itself.

## Using it for practice

**Claude Code (recommended):** clone your fork, run `claude` in the folder, then:

```
/mock-interview technical
/mock-interview market-view 6
/mock-interview mental-maths 10 hard
/mock-interview mixed
```

The skill reads the latest brief and the question bank, asks one question at a time, scores
each answer 1-5, gives the model answer and the pushback, and ends with a summary. With
`OILPRICEAPI_KEY` exported in your shell, the OilPriceAPI MCP server in `.mcp.json` lets it
pull live oil and gas prices during market-view questions.

**claude.ai chat (no setup):** open `.claude/skills/mock-interview/SKILL.md`, copy the
"paste-into-chat" block at the bottom into a new chat, then paste the latest brief and the
question file you want. For live quotes in chat, connect the **FMP** or **Alpha Vantage**
connector from the claude.ai connector directory.

## Running the scripts locally

```
pip install -r requirements-dev.txt
python -m pytest tests -q                 # spread maths and pipeline tests, no network
python scripts/fetch_prices.py            # Yahoo futures + spreads -> data/prices-DATE.{json,md}
python scripts/fetch_news.py              # headlines by sector -> data/news-DATE.json
python scripts/build_brief.py             # -> briefs/DATE.md
```

`build_brief.py --tier1` picks the analysis-tier backend:

| Value | What it uses |
|---|---|
| `auto` (default) | the `claude` CLI if it is on your PATH, else `ANTHROPIC_API_KEY`, else Tier 0 only |
| `claude-code` | the `claude` CLI, on your Claude subscription. No API key; `ANTHROPIC_API_KEY` is stripped from the child process so a stray key cannot move the run onto API billing |
| `api` | the Anthropic API via `ANTHROPIC_API_KEY` |
| `off` | data-only Tier 0 (same as `--no-ai`) |

So with the CLI installed and logged in (`curl -fsSL https://claude.ai/install.sh | bash`, then `claude`), a plain `python scripts/build_brief.py` writes a full analysis-tier brief on your subscription with no key set anywhere. Tier 1 never loses the data: if the backend fails, the Tier 0 brief is written with the error in an HTML comment.

Prices are Yahoo Finance futures, about 15 minutes delayed, and are indicative only. Official
LME prices are not free; COMEX copper is used as the proxy and labelled as such.

## Maintaining it

- **Change what the brief covers:** edit the instrument table at the top of `scripts/fetch_prices.py` and the sector queries at the top of `scripts/fetch_news.py`.
- **Change how the brief reads:** edit `prompts/brief-analysis.md`. The template headings are used by the mock-interview skill, so keep them.
- **Change the schedule:** the cron in `.github/workflows/daily-brief.yml` is in UTC. `30 5 * * *` is 06:30 BST; switch to `30 6 * * *` after clocks go back on 25 Oct 2026 if you want it to stay at 06:30 UK time.
- **A ticker stops working:** the brief shows `n/a` and lists the error at the bottom of the prices table. Update the ticker in the instrument table.
- **Change the dashboard:** edit `docs/index.html`; pushing it redeploys Pages. Preview locally with the `--inline` command above and open `site/preview.html`.
- **Pause it:** Actions tab, select the workflow, "Disable workflow". Or delete the `schedule:` block. Same for the dashboard workflow.
- **Cost:** the free tier is free. The analysis tier runs against your Claude subscription through `CLAUDE_CODE_OAUTH_TOKEN` (one run a day, counted as normal Claude Code usage), or against an API key if you set `ANTHROPIC_API_KEY` instead, which costs a few cents to a few tens of cents a day depending on the model.

## Data sources

| Layer | Source | Key | Freshness | Covers |
|---|---|---|---|---|
| Baseline | Yahoo Finance via `yfinance` | none | ~15 min delayed exchange futures | Brent, WTI, RBOB, ULSD, Henry Hub, TTF, COMEX copper, aluminium, gold, wheat, corn, EUR/USD, DXY, US 10y |
| Overlay | [OilPriceAPI](https://www.oilpriceapi.com) REST and MCP | free key (~50 requests/day) | source-timestamped | Brent, WTI, Henry Hub, TTF, JKM, coal, EU carbon, gold |
| Headlines | Google News RSS, GDELT | none | live | all sectors plus Glencore and other trading houses |
| Analysis | Claude with web search | your Anthropic key | live | fills gaps (Dubai, LME metals, cobalt) with citations, writes the analysis |
| Chat quotes | FMP or Alpha Vantage connectors in claude.ai | free plan | FMP near-live, Alpha Vantage daily | futures and spot quotes during chat practice |
| Not wired in | MetalMiner MCP (paid) | membership | live | official LME copper, aluminium, zinc, nickel, cobalt |

## Handover note

This repo was built to be handed over. After forking, the original owner should disable the
workflow on their copy so only one pipeline runs. All keys, Actions minutes and costs then
live in the fork.
