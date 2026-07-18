# kongfz-book-monitor

Scrape and monitor second-hand listings on https://www.kongfz.com/ for a target list of ISBNs, filter by a desired discount rate, and render a GT report.

Designed to be run periodically (manual or cron) — "refresh, see what's cheap, buy, repeat."

## What it does

1. Sends a keyword search (by ISBN) against Kongfz's `pc-gw` JSON API for each book in the watchlist.
2. Extracts per-listing details: price, shipping, quality, sold-out status, seller link.
3. Computes `total_price = price + shipping`, compares against `full_price × target_discount`.
4. Produces `monitor-books.html` (GT table, markdown-linked book IDs) if any listings hit the discount threshold.

## Watchlist

Defined at the top of `monitor.R`:

```r
isbn_numbers <- c("9787502981464", "9787305281334", "9787301323748")
full_price   <- c(88, 79.8, 59)   # publisher / full retail price
DESIRED_DISCOUNT_RATE <- 0.40     # 40% of full price
```

Add/remove ISBNs in pairs — `tibble()` will complain if lengths mismatch.

## Output

- **Match found** → `monitor-books.html` written to project root via `gt::gtsave()`, console: `"GREAT - TARGET BOOK FOUND."`
- **No match** → only a console message, no artefact (intentional — see *Known issues*).

Columns surfaced: `book_isbn`, `book_total_price`, `current_discount_rate`, `target_discount`, `book_price`, `book_shipping_fee`, `book_full_price`, `book_quality`, `book_quality_text`, `book_show_time`, plus a markdown-linked `book_id`.

## Dependencies

```r
install.packages(c("httr2", "dplyr", "purrr", "gt", "stringr", "here"))
```

R >= 4.3 recommended (uses native `|>` pipe).

## Running

```r
source("monitor.R")
# or
Rscript monitor.R
```

Each ISBN is throttled (`Sys.sleep(3)`) to avoid hammering the API. A full run of 3 ISBNs ≈ 9–12 s.

## Cookie setup (important)

The script currently bakes **live session cookies** into `send_a_web_request()`:

- `shoppingCartSessionId`
- `kfz_uuid`
- `PHPSESSID`
- `kfz-tid`, `kfz_trace`, etc.

These expire in ~1–3 days and are tied to *your* browser session. If the script starts returning empty `data` or 403s, refresh them:

1. Log into kongfz.com in a browser
2. Open DevTools → Network → copy cookies from a `pc-gw` request
3. Paste into the `req_cookies_set()` call

> **Do NOT commit real cookies to git.** Move them to env vars (`~/.Renviron`) before sharing:
> ```
> KFZ_COOKIE_SHOPPING_CART=...
> KFZ_COOKIE_UUID=...
> ```

## Known issues / TODO

- **Keyword search, not ISBN-specific** — the `pc-gw/.../keyword/list` endpoint can return false positives (title matches, different editions). A dedicated `/isbn/` or `/product/` endpoint would be safer.
- **No error handling** on `req_perform()` — one 404/429 kills the whole loop (fix: `tryCatch` or `req_retry`).
- **`mutate_all(as.character)` nukes column types** — keeps GT happy but makes downstream filtering fragile. Should be removed.
- **No artefact on "no match"** — cron job has no signal it ran. Consider touching a sentinel file or writing an empty HTML.
- **Hardcoded `userArea=1006000000`** — locks search to a single region. Make it a parameter if you travel / compare regions.
- **Cookies + `here::here()`** — `here` not declared as a library() but used; add it.

## Next-level ideas

- Swap `for` + pre-allocation → `purrr::pmap` over the watchlist tibble (cleaner, free `.progress`).
- Replace `Sys.sleep(3)` with `httr2::req_throttle()`.
- Wrap in a `targets` pipeline if the watchlist grows beyond ~10 ISBNs (caching + DAG + persisted `_targets/objects`).
- Add a "best price per ISBN" `slice_min()` and email/notify hook.

## License

Personal / educational use. Kongfz ToS applies — don't hammer.
```

---

Couple of calls on tone:
- I kept the cookie warning upfront because that's the thing that'll break first and also the thing you *don't* want leaking to GitHub.
- The "Known issues / TODO" section doubles as a roadmap — if you ever come back to this in 3 months, you'll know exactly what you'd flagged in the peer review without re-reading the diff.
- I didn't over-document the `extract_book_details()` internals — the code is readable, README should stay at the "how do I use/extend this" level.