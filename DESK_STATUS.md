# Catalyst Desk — VM status (auto-generated, Code -> Fable)

Updated 2026-10-15 00:52Z. Raw files: https://raw.githubusercontent.com/yaronkope/catalyst-prices/main/<file>

| file | age (min) | expected ≤ | state | detail |
|---|---|---|---|---|
| prices.json | 1 | 30 | ✅ | 193/195 quotes; 0 errors |
| candidates.json | 5 | 45 | ✅ | 51 movers, market closed; 2 errors |
| news.json | 11 | 45 | ✅ | 251 items, sources ok 17/21; 4 errors |
| gnews.json | 108 | 180 | ✅ | 182 entities; 0 errors |
| closes.json | 278 | 1800 | ✅ | 45 sessions; 2 errors |
| calendar.json | 1309 | 1560 | ✅ | 148 earnings (95 confirmed), 7 macro; 0 errors |

**news.json sources down:** `edgar_fts` first order: HTTP 403; `prnewswire_rss` HTTP 404; `stocktitan_filings` HTTP 429; `stocktitan_news` HTTP 429

## Notes from Claude Code

**2026-10-11 22:50 IL**
- §0 done: `news.json` = build_news.py, tier 1 every 20 min (:02), tier 2 at 10:07/14:07/20:07 IL. First live run: 281 items, 28/36 sources ok. You can switch the scans on.
- Down: `edgar_fts` (SEC blocks full-text search from cloud IPs, also via Jina; not a payment issue). Workaround available: the SEC current-8-K Atom feed works from the VM (no phrase search). `stocktitan_*` and `capitoltrades` 429, `pentagon_contracts` / `iren` / `ferc` 403, `coreweave` regex. Next pass will tune these.
- `gnews.json` keeps the Google News counts (`count_24h` vs `avg_daily_30d`); candidates.json uses it for `news_velocity`.
- §1–4 live (prices +ret_5d/mcap_usd, closes, candidates, calendar). calendar: confirmed = Yahoo info shows the same date with isEarningsDateEstimate=false (88/147). Bank of Israel: waiting for your yaml dates.
- §5–6 not started.

## How to reach Claude Code

Put requests in a file named `to_code.md` in the artifact https://claude.ai/artifact/5BtbfKaqkkTnkcBZP8yVER
(newest first, dated). Yaron only has to tell Claude Code "check Fable"; answers appear in this file.
