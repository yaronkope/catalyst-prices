# Catalyst Desk — VM status (auto-generated, Code -> Fable)

Updated 2026-10-30 15:37Z. Raw files: https://raw.githubusercontent.com/yaronkope/catalyst-prices/main/<file>

| file | age (min) | expected ≤ | state | detail |
|---|---|---|---|---|
| prices.json | 1 | 30 | ✅ | 193/193 quotes; 0 errors |
| candidates.json | 10 | 45 | ✅ | 102 movers, market open; 0 errors |
| news.json | 16 | 45 | ✅ | 341 items (219 tagged = 64%), sources ok 21/21; 0 errors |
| gnews.json | 93 | 180 | ✅ | 182 entities; 0 errors |
| closes.json | 1102 | 1800 | ✅ | 45 sessions; 0 errors |
| calendar.json | 694 | 1560 | ✅ | 101 earnings (76 confirmed), 6 macro; 0 errors |

## Notes from Claude Code

**2026-10-11 22:50 IL**
- §0 done: `news.json` = build_news.py, tier 1 every 20 min (:02), tier 2 at 10:07/14:07/20:07 IL. First live run: 281 items, 28/36 sources ok. You can switch the scans on.
- Down: `edgar_fts` (SEC blocks full-text search from cloud IPs, also via Jina; not a payment issue). Workaround available: the SEC current-8-K Atom feed works from the VM (no phrase search). `stocktitan_*` and `capitoltrades` 429, `pentagon_contracts` / `iren` / `ferc` 403, `coreweave` regex. Next pass will tune these.
- `gnews.json` keeps the Google News counts (`count_24h` vs `avg_daily_30d`); candidates.json uses it for `news_velocity`.
- §1–4 live (prices +ret_5d/mcap_usd, closes, candidates, calendar). calendar: confirmed = Yahoo info shows the same date with isEarningsDateEstimate=false (88/147). Bank of Israel: waiting for your yaml dates.
- §5–6 not started.

**2026-10-18 17:30 IL — fix list of 18.10 executed (Claude Code on the VM)**
- **1. Tagging — DONE.** `aliases.json` is at `/opt/catalyst/aliases.json` (edit it there; `--aliases` flag in build_news.py). Rules (a)–(d) as specified, plus guards found on the live feed: single-word aliases that are ordinary words (apple, meta, intel, spy, crude, micron…) only when Capitalized/ALL-CAPS; ≤3-char aliases (abb, aws, wti, j&j) only as uppercase tokens; `RTX` not when followed by a model/product word (RTX 5090 / RTX Spark are GPUs); URLs and publisher boilerplate ("Add Yahoo as a preferred source … on Google") stripped before matching. Deliberately skipped aliases: `שקל`/`shekel`/`nis` (shekel amounts are not USD/ILS news), `טבע`, `youtube` (links in Truth posts). Explicit tickers from a source (stocktitan URL/`| XYZ Stock News`, `$cashtag`, `(Nasdaq: XYZ)`, SEC CIK map) are kept even outside the 195-symbol universe; a bare uppercase token still needs the universe (≥3 letters, not a stopword). `tickers_resolved` = `tickers`. Offline re-tag of the 48h window: 131/228 = 57.5% (was 26/228); live number is in the table above.
- **2. edgar_fts — fallback LIVE.** `efts.sec.gov` returns `HTTP 403 {"message":"Forbidden"}` for *every* client, including a residential Mac with a browser UA and even the `/LatestSearch/` UI page — SEC closed the public full-text API, it is not an IP/UA problem. The body is logged in `sources.edgar_fts.error` as you asked. The source now falls back to the current-8-K Atom feed on www.sec.gov (works, 200) and greps the primary 8-K + up to two EX-99 exhibits of filings with Items 1.01/1.02/2.01/7.01/8.01 for the same 8 phrases; the Item 1.01/1.02 headings are removed first (they contain "definitive agreement"). 1 req/s, ≤40 new filings per run, cache `state/edgar_scan.json`. `ok:true`; items depend on what was filed (weekend = few).
- **3. stocktitan — rate-limit fix LIVE.** Alone, `/sec-filings/` answers 200 (419 KB) and `/news/` 429 `Retry-After: 7` only when the three requests hit together. Now: the two HTML pages are cached 60 min on disk (`state/stocktitan_*.html`), ≥3 s between stocktitan requests + 2–4 s random delay, Retry-After stored in `state/stocktitan_backoff.json` (stale cache is served during the backoff). Judge after 24h.
- **4. Sub-baskets LIVE in candidates_vm.py.** Benchmarks `cyber_saas`, `rare_earth`, `telecom`, `defense_primes` are equal-weight baskets (`benchmarks[name].kind = "equal_weight"`, `members` listed); RIOT/MARA/CIFR moved to `crypto_linked` (BTC-USD), OKLO+LEU to `uranium` (URA), the primes out of `defense_drones`. Basket members: `dev_group = excess_1d`. Replay `--date 2026-10-16`: 9/12 cyber/SaaS names `beta` vs `cyber_saas` (bench −7.7%); FSLY/OKTA/ESTC idiosyncratic at |excess| ≥ 2%.
- **5. BoI dates:** untouched, waiting for the pasted list.
- **6. CTRA / VRNT cannot be added:** Yahoo has no CTRA at all — Devon closed the Coterra merger on 7 May 2026 (0.70 DVN per CTRA, delisted). VRNT last traded 25 Nov 2025 (Thoma Bravo take-private). Both removed from `tickers.txt`; `DVN` is already in the list as the successor. Please drop them from your side (aliases.json / cloud list).

## How to reach Claude Code

Put requests in a file named `to_code.md` in the artifact https://claude.ai/artifact/5BtbfKaqkkTnkcBZP8yVER
(newest first, dated). Yaron only has to tell Claude Code "check Fable"; answers appear in this file.
