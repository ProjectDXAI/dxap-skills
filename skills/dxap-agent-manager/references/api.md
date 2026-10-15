# Hosted management API

All paths below are relative to the account REST base in [setup](setup.md).
REST fields use camelCase. Use the
current [OpenAPI schema](https://api.dxap.ai/openapi.json) for complete shapes.
This reference covers the operations used by this skill.

## Configuration and directives

| Method and path | Purpose and useful results | Scope |
| --- | --- | --- |
| `GET /agents` | Discover `agents`, IDs, names, `runtimeMode`, status, and selected accounts | `read:account:agents` |
| `GET /agents/{agentID}` | Read configuration, `promptBlocks`, `activeDirectives`, `selectedExecutionAccount`, and `updatedAt` | `read:account:agents` |
| `GET /agents/{agentID}/config-history` | Read `revisions`, changed fields, diffs, and optional snapshots | `read:account:agents` |
| `GET /agent-models` | Discover selectable model IDs | `read:account:agents` |
| `GET /prompt-templates` | Discover selectable template references | `read:account:agents` |
| `POST /agent-prompt-preview` | Render proposed prompt configuration for inspection; does not save settings or run a turn | `read:account:agents` |
| `PATCH /agents/{agentID}` | Update supplied configuration fields; read details again to verify | `write:account:agents` |
| `GET /agents/{agentID}/directives` | Read active `directives`, lifecycle state, remaining counts, and `asOf` | `read:account:agents` |
| `POST /agents/{agentID}/directives` | Create future behavioral guidance; returns its ID | `write:account:agents` |
| `DELETE /agents/{agentID}/directives/{directiveID}` | Cancel the selected directive; returns 204 | `write:account:agents` |
| `GET /agents/{agentID}/trading-memory` | Read supporting trading memories when needed to explain behavior | `read:account:agents` |

Configuration history accepts `limit`, `before`, and `includeSnapshots`. Use the
returned `nextCursor` as the next request's `before`; do not construct cursors.
Preview has its own request schema: map the relevant editable fields rather than
posting an entire agent response.

## Schedules and monitoring

| Method and path | Purpose | Scope |
| --- | --- | --- |
| `GET /triggers` | List agent schedules and market/position conditions | `read:account:triggers` |
| `GET /triggers/{triggerID}` | Inspect a trigger and linked activity | `read:account:triggers` |
| `POST /triggers` | Create an authorized future schedule or condition | `write:account:triggers` |
| `PATCH /triggers/{triggerID}` | Update the selected trigger | `write:account:triggers` |
| `DELETE /triggers/{triggerID}` | Remove the selected trigger | `write:account:triggers` |
| `GET /execution-accounts` | Read owned accounts and readiness/authorization information | `read:account:execution_accounts` |
| `GET /hyperliquid/activity` | Read turn-level activity with optional child actions and execution evidence | `read:account:actions` |
| `GET /hyperliquid/positions` | Read positions for the selected scope | `read:account:portfolio` |
| `GET /hyperliquid/performance/history` | Read performance observations over the requested period | `read:account:portfolio` |
| `GET /hyperliquid/trades` | Read filled trade history and attribution | `read:account:trades` |
| `GET /hyperliquid/overview` | Read overview context and lifetime agent trading totals | `read:account:overview` |

Examples of focused reads, with an agent ID obtained from discovery:

```text
GET /triggers?agentId=<agentID>&limit=50
GET /hyperliquid/activity?agentId=<agentID>&range=24h&includeChildren=true&limit=50
GET /hyperliquid/positions?agentId=<agentID>&executionAccountId=<accountID>&limit=50
```

Trigger responses include `type`, `status`, `condition`, `disabledReason`,
`nextFireAt`, `lastFiredAt`, `lastTurnId`, and optional `lastActivity`. Find
schedules by their returned `type: schedule`. Paginate the agent's trigger list;
do not assume the schedule is on the first page or invent a `type` query filter.

Activity returns `activity`, `pageInfo`, and `asOf`. With `includeChildren=true`,
inspect child status, intent, execution, and fill evidence. Trigger and activity
pages use `pageInfo.nextCursor` as `cursor` while `hasNextPage` is true. Check
each other route's schema for its pagination and accepted filters. In particular,
account-wide summaries must not be presented as agent-specific observations.

Use bounded periods and page limits. A complete answer may need several pages;
report incomplete coverage when stopping before the requested period is covered.
HTTP success confirms the request, not a trading outcome. A timed-out write has
an uncertain result: follow [management recovery](management.md#verify-and-recover).
