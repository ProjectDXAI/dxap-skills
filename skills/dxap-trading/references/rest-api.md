# External trading REST API

Use this reference for a REST client or to map MCP actions to HTTP routes.
This interface requires an external agent created on DXAP and a credential bound
to it and its selected account. It cannot trade for a hosted agent.
The base is `https://api.dxap.ai/api/v1/account/external-agent`.
Authenticate with `Authorization: Bearer <API key>` and use `Content-Type:
application/json` for JSON requests. The credential supplies the bound agent,
account, and paper/live mode.

Consult the [OpenAPI schema](https://api.dxap.ai/openapi.json) for exact
shapes. The table uses scope suffixes from [setup](setup.md); for example,
`context` means `read:external:context`, and `trades` means `write:external:trades`.
Use the connection address configured for the user's DXAP account.

## Routes

| HTTP method and path | MCP tool | Scope |
| --- | --- | --- |
| `GET /context` | `get_account_context` | context |
| `GET /summary` | `get_trading_summary` | context + history; market_data when requested |
| `GET /portfolio` | `get_portfolio` | context |
| `GET /policies` | `get_trading_limits` | context |
| `GET /market-context` | `get_market_overview` | market_data |
| `GET /performance` | `get_account_performance` with `limit: 1` | context |
| `GET /performance/history` | `get_account_performance` | context |
| `GET /activity` | `list_actions` | history |
| `GET /requests/{clientActionID}` | `get_action_receipt` | history |
| `GET /orders/{orderID}` | `get_order` | context |
| `POST /orders/market` | `submit_perp_market_order` | trades |
| `POST /orders/limit` | `submit_perp_limit_order` | trades |
| `POST /orders/modify` | `modify_perp_limit_order` | orders |
| `POST /orders/cancel` | `cancel_perp_limit_order` | orders |
| `POST /positions/exits` | `set_perp_exit_orders` | orders |
| `POST /positions/exits/cancel` | `cancel_perp_exit_orders` | orders |

`/performance` has OpenAPI operation ID `get_latest_account_performance` and
returns at most one valuation. `/performance/history` accepts a limit and uses
`get_account_performance`. The other operation IDs match the MCP names above.
MCP names may be namespaced by the harness; resolve them through its tool catalog.

## First read

With `DXAP_API_KEY` already supplied through the harness's secret environment:

```sh
curl --fail-with-body --silent --show-error \
  'https://api.dxap.ai/api/v1/account/external-agent/summary?action_limit=10' \
  --header "Authorization: Bearer ${DXAP_API_KEY}"
```

The summary requires context and history scopes. With context access only,
request `/context` instead. No read submits an order.

For selected market observations, use:

```text
GET /summary?action_limit=10&include_market=true&symbols=BTC,ETH
```

The corresponding MCP arguments are:

```json
{"action_limit": 10, "include_market": true, "symbols": ["BTC", "ETH"]}
```

The summary includes `context`, `recent_actions`, optional `market`, and
`assembled_at`. All requested sections must succeed. The sections can describe
different observation times; check those times before combining their findings.

Symbol lists accept at most 20 entries. REST uses a comma-separated `symbols`
query parameter; MCP uses an array. Matching is case-insensitive and duplicates
are combined. Preserve prefixes such as `xyz:SP500`. An omitted or empty list
selects all available markets; unknown symbols produce no matching markets.
Summary symbol filters require `include_market=true` and affect market data only.

## Action bodies

REST and MCP use the same snake-case command fields. Decimal values are strings;
`leverage` and `expected_revision` are integers. Use the current schema for each
operation; extra fields from other operations may be rejected.

This is an illustrative paper market-open body, not an instruction to trade.
Generate and persist your own fresh action UUID for a new action. The credential
determines whether submission would be paper or live:

```json
{
  "client_action_id": "a8dfc715-28b7-45b0-805d-a911b98087fb",
  "symbol": "BTC",
  "operation": "open",
  "direction": "long",
  "margin_usd": "25",
  "leverage": 1,
  "reason": "User-authorized paper integration check"
}
```

Send an authorized open to `POST /orders/market`, or add `limit_price` and use
`POST /orders/limit`. A close uses `operation: "close"`, the position direction,
and `close_percent` instead of margin/leverage. Market and limit entry orders
accept optional `take_profit_price` and `stop_loss_price`; their activation differs
as explained in [order lifecycle](order-lifecycle.md).

Modify/cancel bodies need `order_id` and `expected_revision`. Position exit
commands need `symbol`. Every new mutation needs its own `client_action_id`.
An exact retry preserves all fields, including `reason`, and the same ID.
`reason` is optional, at most 2,000 characters; symbols and top-level decimal
strings are at most 64 characters.

## History and response handling

- Activity defaults to 20 actions, maximum 100. Use the last `request_id` as
  `before`. A summary's `recent_actions.next_cursor` is also an activity cursor;
  `recent_actions.has_more` indicates older actions.
- Performance history defaults to 100 observations, maximum 500, newest first.
  Use the oldest returned `observed_at` timestamp as `before`. Check `complete`
  and the observation time before using a valuation.
- REST activity and performance routes return arrays. MCP wraps these arrays in
  `actions` and `points`, respectively. REST `/performance` also returns an array
  with at most one item.
- Read a receipt using the original caller's `client_action_id`. The `request_id`
  used for activity pagination is a different identifier.
- An HTTP success or `recorded` receipt alone does not prove execution or a fill.
  Inspect receipt, execution, and order states together. MCP can also return a
  tool error even when transport succeeded.
- Preserve error bodies and action IDs. A conflict may indicate different inputs
  under an existing ID, a changed order revision, or an unresolved earlier action.
  Read the cause and current state before changing a command.
- A timeout or temporary error after submission requires receipt recovery.
  Follow [order lifecycle](order-lifecycle.md) instead of blind automatic resends.
