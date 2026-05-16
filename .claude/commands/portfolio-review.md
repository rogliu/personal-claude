---
description: Run a book-aware investment review — debate verdict(s) passed through a portfolio-level overlay against your actual holdings
---

Run a portfolio-aware review for: **$ARGUMENTS**

`$ARGUMENTS` is one of:
- a single ticker (e.g., `IBIT`) — review that one position/candidate against the book
- `holdings` — review every debatable position in `portfolio.json` (skip pure index/target-date/cash sleeves: FXAIX, SSGA_SP500_K, PDHM, PEYL, USD, SPAXX — these are core ballast, not debate candidates)
- a comma list (e.g., `NVDA,GOOGL,BTC`)

## Step 0 — Load the book (required first step)

Read `.claude/portfolio.json`. If it does not exist, tell the user to create it (or paste holdings) and stop. Capture it as **PORTFOLIO**. Note `totalValue`, `policy`, and `lookThroughNotes`.

## Step 1 — Asset verdict(s)

For each ticker under review, obtain a structured debate verdict. Prefer freshness:
- If a verdict for this ticker was produced earlier in the current session, you may reuse it — but state the date and that it is reused.
- Otherwise run the full `/debate` flow for that ticker (dossier → R1 → challenger → revised mediator) and capture the **revised mediator verdict** (near-term + long-term + Overall Action + confidence + single most important factor + key unresolved questions).

Map crypto sensibly: a holding like `IBIT` or `ZCSH` should be debated as the underlying (`BTC`, `ZEC` / Zcash) since that drives it.

## Step 2 — Portfolio overlay

Invoke the `portfolio-manager` subagent **once**, passing it:
- the full PORTFOLIO json
- every asset verdict gathered in Step 1, clearly labeled

Capture its output as **OVERLAY** (sized actions table + top-3 book moves + single most important portfolio factor).

## Step 3 — Track-record ledger

Append one JSON object **per asset reviewed** to `.claude/verdict-ledger.jsonl` (create the file if absent; one compact JSON object per line, no array wrapper):

```
{"date":"<today>","ticker":"<t>","price_or_level":"<from dossier §1 if available else n/a>","near_term":"<BUY/SELL/HOLD>","long_term":"<BUY/SELL/HOLD>","overall":"<BUY/SELL/HOLD>","confidence":"<Low/Med/High>","portfolio_action":"<EXIT/TRIM/HOLD/ADD/INITIATE>","current_weight_pct":<num>,"single_most_important_factor":"<one line>","rounds_run":<n>,"challenger_moved":"<YES/NO>"}
```

This is the accountability spine — every call is dated and price-stamped so a future `/scorecard` can grade it. Never edit or delete prior lines; append only.

## Output order (strict)

1. **PORTFOLIO snapshot** — total value, current weights of reviewed assets, policy limits, look-through note.
2. For each reviewed ticker: the **debate verdict** (full, or a tight summary if reused — and say which).
3. **OVERLAY** in full (the portfolio-manager output).
4. Confirm the ledger line(s) appended (echo them).
5. **Bottom line** (last thing shown):

- **Book**: total value, dominant risk cluster
- **Per-asset sized actions**: ticker → current wt → portfolio action → $ band
- **Top 3 book-level moves** (ranked)
- **Single most important portfolio factor**
- **Policy breaches** (if any)
- **Reminder**: decision-support only, not financial advice, you execute every trade.

Keep the final bottom line tight and scannable — it is the deliverable.
