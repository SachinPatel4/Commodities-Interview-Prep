# Handover

This toolkit is designed to be owned and run by one person with no help from whoever built it.
Follow this once and it runs itself.

## For the person handing it over (Alistair)

1. Transfer the repo: Settings, General, Danger Zone, "Transfer ownership", enter Sachin's GitHub username. He gets an email and accepts. History, branches and workflows move with it.
   (A fork works too if you want to keep a copy. If you keep any copy, disable both workflows on it so only one pipeline runs.)
2. Send Sachin the link to this file. Nothing else is needed from you.

## For the new owner (Sachin)

Do these in order. About 15 minutes.

1. **Make the repo public.** Settings, General, Danger Zone, "Change visibility", Public. GitHub Pages and unlimited Actions minutes only work on public repos on the free plan. Nothing secret is in the repo: keys go in secrets, which are never visible.
2. **Rename the default branch to `main`** (optional but tidy): Settings, General, Default branch, pencil icon.
3. **Enable Actions**: Actions tab, "I understand my workflows, go ahead and enable them" if asked.
4. **Add secrets**: Settings, Secrets and variables, Actions, "New repository secret":
   - `CLAUDE_CODE_OAUTH_TOKEN`. Turns on the analysis tier of the brief (five stories, why they matter, questions, view of the day) and bills it to your Claude subscription, not an API key. Install the CLI with `curl -fsSL https://claude.ai/install.sh | bash`, run `claude setup-token`, approve in the browser, and paste the printed token in as the secret. Lasts a year; needs a Pro, Max, Team or Enterprise plan. Without it you still get prices, spreads and headlines.
   - `ANTHROPIC_API_KEY` from https://console.anthropic.com is the alternative, billed as API usage at a few cents a day. Set one or the other, not both.
   - `OILPRICEAPI_KEY` from https://www.oilpriceapi.com/auth/signup (free). Adds JKM, coal, EU carbon and source-timestamped oil and gas prices.
5. **Enable Pages**: Settings, Pages, Source = "GitHub Actions".
6. **Run both workflows once**: Actions tab, "Daily commodities brief", "Run workflow"; then "Dashboard (GitHub Pages)", "Run workflow". Two minutes later:
   - the brief is at `briefs/<today>.md` in the repo;
   - the dashboard is at `https://<your-username>.github.io/<repo-name>/`.
7. **Bookmark the dashboard on your phone.** It rebuilds after each daily brief and hourly on weekdays.

## Every day

- 06:30 UK: the brief lands in `briefs/` and the dashboard updates. A retry runs at 09:00 UK if GitHub skipped the first slot.
- If a day is missing, press "Run workflow" on "Daily commodities brief". A manual run replaces that day's brief; scheduled runs never overwrite one.
- Read the brief, then run a mock interview:
  - **Dashboard, Interview tab**: paste your Anthropic API key once in Settings (kept in your browser only). One question at a time, scored out of 5, with the model answer and the pushback.
  - **Claude Code**: clone the repo, run `claude`, then `/mock-interview technical` (or `market-view`, `behavioural`, `mental-maths`, `mixed`).
  - **claude.ai chat**: copy the paste-in block at the bottom of `.claude/skills/mock-interview/SKILL.md`, then paste the latest brief and a question file.

## Publishing the dashboard inside claude.ai (optional)

In Claude Code, in the repo folder, ask Claude to run
`python scripts/build_dashboard_data.py --inline docs/index.html --out site/preview.html`
and publish `site/preview.html` as an artifact with the `sample` capability. In that copy the
interviewer marks answers through your claude.ai account and needs no API key. It is a snapshot;
republish when you want fresh numbers. The Pages version refreshes itself.

## Changing things

- Instruments and spreads: the table at the top of `scripts/fetch_prices.py`.
- Headline searches: the sector list at the top of `scripts/fetch_news.py`.
- How the brief reads: `prompts/brief-analysis.md`.
- Questions: `questions/*.md`, same format as the existing entries.
- Company facts for networking: `networking/tuesday-cheatsheet.md`.
- Dashboard: `docs/index.html`. Pushing it redeploys Pages.
- Schedule: cron lines in `.github/workflows/daily-brief.yml` (UTC). Move them an hour later after 25 Oct 2026 to stay at UK times.

## Pausing or stopping

Actions tab, pick the workflow, three-dot menu, "Disable workflow". Do it for both workflows.
Delete the two secrets if you want to be sure nothing can spend money.

## If something breaks

- A red run: open it in the Actions tab; the failing step prints the error. Most likely causes: a Yahoo ticker changed (edit the instrument table), or a key expired (replace the secret).
- Prices show `n/a`: the errors list at the bottom of the prices table names the ticker.
- Pages 404: Pages is not enabled or the repo is private.
- Ask Claude Code in the repo folder; `CLAUDE.md` tells it how everything fits together.
