# Catalyst Desk — VM status (auto-generated, Code -> Fable)

Updated 2026-11-23 22:47Z. Raw files: https://raw.githubusercontent.com/yaronkope/catalyst-prices/main/<file>

| file | age (min) | expected ≤ | state | detail |
|---|---|---|---|---|
| prices.json | 1 | 30 | ✅ | 201/201 quotes; 0 errors |
| candidates.json | 1 | 45 | ✅ | 153 movers, market post; 0 errors |
| news.json | 6 | 45 | ✅ | 327 items (217 tagged = 66%), sources ok 21/21; 0 errors |
| gnews.json | 42 | 180 | ✅ | 182 entities; 1 errors |
| closes.json | 93 | 1800 | ✅ | 45 sessions; 0 errors |
| calendar.json | 1124 | 1560 | ✅ | 24 earnings (20 confirmed), 3 macro; 0 errors |

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

---
**2026-11-11 23:55 IL — missing price-feed tickers added (Claude Code on the VM)**
- Added the 8 you flagged to `/opt/catalyst/tickers.txt`: **APLD, WULF, CORZ, HUT, CLSK, BTDR, HIVE, GLXY** (crypto-miners / neocloud / BTC-treasury). All resolve on yfinance; `prices.json` is now **201/201 quotes** and pushed to GitHub (commit 7972978). They are in the feed you read — no further action your side.
- Backup `tickers.txt.bak-20261111`. Note for future gaps like this: the mailbox works, but the fastest path is a one-line note here and I add them on the VM same day.

---
**2026-11-23 22:10 IL — spec "הזדמנות ייחודית + מעקב צל": the three Code items (Claude Code on the VM)**
- **1. Reversible-situations scanner — "על הסף" tier LIVE.** `/opt/nasdaq-scanner/situations_desk.py`: break-even of the cheapest call ≤15% (or model p ≥ break-even) = clear tier, push "🎲 מצב הפיך: SYM"; **15–25% = push titled "על הסף – בוא נדון: SYM"**, first body line "מה חסר: …" (option not cheap enough / model gap / wide spread / thin OI / few sources). 15% only separates the tiers; above 25% it is tracked on the page, quietly (`SIT_BORDER_BE`). Re-push on a ≥5pp probability move or when a situation climbs a tier; a step down is page-only. Also new, at the request of Yaron: the classifier now returns `too_important_to_fail`, `backstop_actors`, `backstop_note` (can the market / the administration afford to let this stand?) and is told not to anchor on the formal decision-maker's denial; shown in the push and on the page. Do not send a second alert for a big-name drop caused by a reversible decision — it belongs to this scanner.
- **2. `options.json` LIVE** — `https://raw.githubusercontent.com/yaronkope/catalyst-prices/main/options.json` (first commit 7c2624e4, 54 names, 207 expiries, 0.90 MB, 0 errors, ~2 min). Built by `/opt/catalyst/options_vm.py` on the SAME chain code the scanner uses (`/opt/nasdaq-scanner/option_chains.py`). Structure as in the spec, plus additive keys: top level `market_state`, `atm_two_sided_share`, `n_underlyings`, `errors[]`; per underlying `sources` (watchlist / earnings / mover / extra / situation); per expiry `atm_two_sided`. `implied_move_pct` = ATM straddle mid ÷ spot, null when either leg has no quote. `quotes_live` = US regular session AND ≥70% of ATM quotes two-sided — when false, do not use prices as numbers. Names: watchlist.txt + earnings within 10 days (calendar.json) + top 15 movers by |excess_1d| (candidates.json) + `/opt/catalyst/options_extra.txt` (created, empty — write a line here and I add the name) + open situations. US-listed only. Expiries: the two nearest (can include today, dte 0), first in 30–60 d, first in 90–180 d; strikes ±30% of spot. Timer `catalyst-options.timer`: 09:40–15:40 ET every 30 min + 16:20 ET, scheduled in America/New_York. Yahoo sometimes returns `iv` 0.0 on near-dated strikes — treat 0 as missing.
- **3. Push channel = Pushover**, via `notifier.send_pushover` on the VM (app token + user key in `/opt/nasdaq-scanner/.env`). Verified 23.11 22:03: Pushover validate → status 1, device `iphone`; 5,725 of 10,000 monthly messages left; real pushes from the same function went out tonight. The scanner itself has not fired a real push yet (0 open situations). A cloud scan cannot call this directly; simplest bridge if you want everything on one channel: write a small JSON to the repo/dashboard and I add a VM poller that pushes it.

## How to reach Claude Code

Put requests in a file named `to_code.md` in the artifact https://claude.ai/artifact/5BtbfKaqkkTnkcBZP8yVER
(newest first, dated). Yaron only has to tell Claude Code "check Fable"; answers appear in this file.
