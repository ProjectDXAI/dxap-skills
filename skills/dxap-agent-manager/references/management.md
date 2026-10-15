# Settings, directives, and schedules

Use this reference before a management write. Preserve the user's authorization
and the hosted-trading boundary in the skill entrypoint.

## Update configuration without losing unrelated values

Fetch current details immediately before preparing a change. Agent PATCH is
sparse at the top level, but supplied `policyConfig`, `tradingConfig`,
`executionPermissions`, and `notificationPreferences` replace those sections.
Supplied `promptBlocks` and `twitterAccounts` replace their lists. Merge the
requested edit into the current section/list and retain unrelated settings.
Send only the changed top-level fields; do not send the whole detail response.

For example, pausing the chosen agent uses `PATCH /agents/{agentID}`:

```json
{"status": "paused"}
```

Changing one limit requires preserving every other current policy field. An
`executionPermissions` update must preserve `allowedTools` when the user only
changes `canTrade`. Retain relevant Twitter-account settings when changing tool
access. Use the current schema's field types; hosted policy USD values are
numbers, not the decimal strings used by external trading commands.

For a strategy edit, preserve unrelated prompt blocks and map them into writable
`title`/`body` fields. Use prompt preview when inspecting the rendered result
would help. Choose models and template references from their current catalogs;
do not invent IDs. `chatPrompt` controls chat instructions and is not a replacement
for trading strategy. An empty `chatPrompt` clears it; omission preserves it.

Policy writes validate the supplied section. In particular, supplied
`maxTradeUsd`, `maxPositionUsd`, `maxNotionalUsdPerRolling24h`, and per-token
rolling notional caps must be at least 15. Existing legacy settings can differ.
If preserving a legacy section makes the requested write invalid, explain the
conflict rather than silently raising caps or dropping restrictions.

There is no client-supplied expected revision on agent PATCH. Compare current
state with the state used to prepare the change, rebase on detected edits, and
read back afterward. This reduces stale writes but cannot eliminate a concurrent
edit between the read and write. Do not claim atomic conflict protection.

## Temporary directives

Directives guide future autonomous behavior; they do not enforce policy limits or
start a turn. Read standing strategy and active directives first to avoid adding
duplicates or conflicting guidance. Use the lifecycle requested by the user.

| Lifecycle | REST fields | Meaning |
| --- | --- | --- |
| `event_count` | `eventType` (`turn` or `trade`), positive `eventCount` | Ends after the specified matching events |
| `open_position` | `positionId` | Ends when the agent's open position is authoritatively observed closed or liquidated |
| `expires_at` | Future RFC3339 `expiresAt` | Ends at the specified time |
| `until_cancelled` | No additional lifecycle fields | Continues until canceled |

Supply `instruction` and `lifecycleType` plus only the relevant lifecycle fields.
Instructions allow at most 4096 UTF-8 bytes. A turn event counts when the turn's
prompt successfully receives the directive, not when it completes. A trade event
counts a successful autonomous perpetual order execution; it is not a count of
chat messages or a guarantee of fills. Reads do not consume events.

Example REST body for `POST /agents/{agentID}/directives`:

```json
{
  "instruction": "For the next three turns, explain any decision to wait in the turn summary.",
  "lifecycleType": "event_count",
  "eventType": "turn",
  "eventCount": 3
}
```

For MCP, add `agent_id` and use `lifecycle_type`, `event_type`, `event_count`,
`position_id`, and `expires_at` instead of the corresponding camelCase fields.
Use a returned directive ID for cancellation. A running turn that already
received the instruction may finish with it after cancellation.

## Schedules, triggers, and pausing

List the agent's triggers before editing its schedule; a schedule is a trigger
with `type: schedule` and a `condition.interval`. Update the existing schedule
instead of creating a duplicate. Supported recurring intervals have a 30-minute
minimum. Preserve the current condition/behavior fields when changing one value.
For market or position conditions, use the current schema and verify the target
agent, account, and position relationship before writing.

A schedule or trigger change can cause future autonomous trading. Use it for
the requested recurring cadence or genuine future condition, never to implement
"trade now" or "force a turn" indirectly. Do not create immediately satisfied
conditions, toggle status, or shorten schedules as a workaround for a forced run.

Pausing the agent stops new autonomous dispatch; it does not promise cancellation
of an already-running turn, accepted orders, or open positions. Hosted
user-approved trades can still be made on the website while paused when trading
permission and account authorization allow them. Disabling `canTrade` is a
separate permission change and also does not cancel accepted orders.

Resuming an agent can make an overdue schedule eligible promptly; missed
intervals are not all replayed. Pausing/resuming a schedule itself can reset its
next firing time. Report the read-back timing rather than promising the old
schedule will be preserved. Never resume an agent solely as a diagnostic probe.

## Verify and recover

Read the affected resource after each write. For configuration, compare the
requested fields and unrelated values in any replaced section; inspect history
when attribution matters. For directives and triggers, retain the returned IDs
and verify their lifecycle, status, and target. Saved settings do not establish
that a subsequent turn used them.

On a timeout or lost response, inspect current state before resending. Directive
creation has no idempotency key: list active directives and look for the intended
instruction and lifecycle. For trigger creation, inspect the agent's trigger
list. Matching content alone is not proof when duplicates or concurrent writes
make attribution ambiguous. Report uncertainty and stop creating more resources
until resolved. Canceling a known directive is safe to repeat.

Do not automatically roll back by overwriting newer settings. Re-read and resolve
conflicting changes. Preserve unresolved outcomes in the harness continuity record.
