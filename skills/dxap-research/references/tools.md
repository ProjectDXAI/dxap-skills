# Research tool guidance

Use the relevant section for the question at hand. Inspect current MCP tool
definitions for the complete schemas and validation rules.

## Find news

Use `market_news_get_signals` for asset developments and macro context. Supply
up to 20 `symbols`, or set `include_macro: true` for a macro-only request.
`lookback_days` accepts 1–30 and defaults to 3. `limit` accepts 1–100 and defaults
to 3 per symbol and for the macro supplement; the overall result is capped at
100 article signal rows.

For recent BTC and ETH developments with macro context:

```json
{
  "symbols": ["BTC", "ETH"],
  "include_macro": true,
  "lookback_days": 1,
  "limit": 5
}
```

For a macro-only brief:

```json
{"include_macro": true, "lookback_days": 1, "limit": 5}
```

Read `articles`, `article_count`, `since`, and `warnings`. Article entries provide
`article_id`, `published_at`, headline, market summary, source type, event type,
relevance, and signal confidence when available. Retain press-release and opinion
labels when judging evidence. Use relevance and confidence to prioritize review;
verify the article before relying on a material claim.

Several rows can refer to the same article for different assets. Group by
`article_id` when assessing independent sources. Empty or limited results describe
the requested coverage, not the full universe of market events.

## Read an article

Use `market_news_get_article` with an `article_id` copied from news signals.
`lookback_days` defaults to 14 and accepts 1–90. If the news search covered an
older period, choose an article lookup window that includes its publication date.
`max_content_bytes` defaults to 12,000 and accepts 1–40,000.

Check `found` before using article details. Read `article.content.text`,
`article.content.truncated`, publication time, signals, and targets. Cite
`article.url` for the article's claims. If content is truncated and more context
is needed, repeat the same lookup with a larger supported content limit.

Preserve the distinction between an article's reporting, an author's opinion,
and DXAP's signal interpretation. Fetching a detail result does not independently
verify the publisher's claims.

## Read curated X commentary

Use `twitter_get_summaries` with required `lookback_hours` from 1 to 24.
Choose `horizon_hours` based on the question:

- `1` (default): completed one-hour summaries within the requested range. Useful
  for comparing how commentary changed across periods.
- `6`: the latest available rolling six-hour synthesis in the requested range.
  Useful for a consolidated view of recent discussion.

The lookback selects summaries to retrieve; the horizon describes how much time
each summary covers. Read `window_start` and `window_end` before attributing a
statement to a particular period.

For BTC and ETH commentary plus broad market context:

```json
{
  "lookback_hours": 6,
  "horizon_hours": 6,
  "assets": ["BTC", "ETH"],
  "include_overall": true
}
```

`assets` accepts up to 10 symbols. With assets, the default is asset-specific
summaries and matching discussion clusters; `include_overall: true` adds broad
context. Omit assets for overall summaries and broad clusters. Setting
`include_overall: false` requires at least one asset.

Read the text content for overall summaries, clusters, asset summaries, and their
sources. The structured fields report `status`, `range`, `horizon_hours`,
`summary_count`, and `missing_assets`. Source entries identify `tweet_id`,
`author_username`, and the source's `relationship` to the claim, including
supporting or contradicting evidence. Preserve that distinction when citing them.

`status: no_data` and missing assets are coverage gaps, not neutral sentiment.
Describe the result as curated commentary for its covered periods. Use the
returned author and tweet identifiers for attribution; do not invent quoted
tweet text from a summary.

## Analyze price action

Use `market_technical_analysis` for one or more perpetual markets and one or more
timeframes. Supply an asset selection such as `symbols` and required `timeframes`.
At most 10 assets can be analyzed per call. Keep market qualifiers such as
`xyz:SP500` when selecting an asset.

```json
{
  "symbols": ["BTC", "ETH"],
  "timeframes": ["1h", "4h"],
  "lookback": 50
}
```

Allowed timeframes are `1m`, `5m`, `15m`, `1h`, `4h`, `1d`, and `1w`.
`lookback` is the number of output candles, default 50 and maximum 500.
Use an RFC3339 `as_of` timestamp when the user requests historical analysis.

Default indicators include SMA, EMA, RSI, MACD, Bollinger Bands, and volume moving
average. Use the tool schema for supported custom indicator settings.

Read the text content for latest/prior indicator values, recent candles,
candle geometry, cross events, coverage, freshness, and signal summaries.
The structured fields include `status`, `result_count`, and `error_count`.
For `partial` results, identify the affected asset/timeframe and explain how it
limits the comparison. For `failed` results, report that the requested analysis
was unavailable. Historical analysis describes its requested time; it is not a
current quote or evidence of an executed trade.
