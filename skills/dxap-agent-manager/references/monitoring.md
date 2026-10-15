# Monitoring and troubleshooting

Use focused reads to answer the user's question. A general agent health check
usually needs details, schedules/triggers, and recent activity; add account,
position, trade, or performance reads when relevant. Paths and scopes are in
[API reference](api.md).

## Establish what was observed

Record the selected agent ID, execution account ID, mode, check time, response
observation times, and coverage window. Activity and historical configuration
can span account changes. Keep agent-wide history distinct from the currently
selected account and from account activity attributable to another source.

Use `GET /hyperliquid/activity` with `agentId`, a bounded `range`, and
`includeChildren=true` when investigating decisions or execution failures.
Inspect turn status, final summary/errors, child actions, and execution/fill
evidence. A completed turn can contain no trade or a rejected action; it is not
proof of successful execution. A resting limit order is not a filled position.

Follow returned cursors when the first page does not cover the requested window.
Empty or unavailable history does not prove that the agent took no action.
Do not substitute a page count for a time window or infer missing order state.

## Explain inactivity or failure

| Evidence | Interpretation and next read |
| --- | --- |
| Agent is draft or paused | Explain its state; do not activate it to test a hypothesis |
| No selected execution account, missing authorization, or disabled trading permission | Report the specific observed restriction; do not enable trading or change the account automatically |
| Model unavailable or disabled | Compare with the current model catalog and report the limitation; do not silently replace it |
| Trigger paused, disabled, or expired | Inspect its condition, status, and `disabledReason`; distinguish it from agent status |
| Schedule `nextFireAt` absent | Check pending/running activity and linked `lastTurnId`; a missing next time can be normal while a scheduled turn is in flight |
| Due time passed | Check observation freshness, pending/running turns, agent/model eligibility, and recent completion before calling it stuck |
| Completed turn with no trade | Read the summary and actions; waiting can be a deliberate decision |
| Failed turn or rejected action | Report the recorded reason, account, and relevant configuration; separate inference failure from an execution rejection |
| Recent configuration change | Inspect configuration history and compare timestamps; correlation alone does not prove the change caused the failure |
| Missing scopes or partial responses | Report which checks could not be made; missing access is not evidence that the agent is unhealthy |

Recurring schedules advance after terminal turn completion, so the interval is
not a promise of exact wall-clock start times. A pending/running scheduled turn
can prevent overlapping scheduled work. A resumed agent can become eligible for
an overdue run. Use the requested monitoring tolerance and observed state rather
than declaring every late or missing timestamp a failure.

For position or performance questions, check valuation completeness and actual
observation times. Agent overview `totalTrades` and `totalVolumeUsd` are lifetime
totals across its accounts, combining paper and live activity; they are independent
of the chart period and selected account and may lag new fills. Do not label them
as today's trades or the selected account's volume.

## Ongoing supervision

When the user asks for recurring monitoring, keep the checking schedule in the
local harness. Do not add DXAP triggers just to wake the supervisor; those affect
the hosted agent's autonomous operation.

Persist enough continuity to compare later observations:

- Target agent/account IDs, expected runtime mode, and authorized management scope.
- Check cadence, requested time window, notification criteria, and lateness tolerance.
- Last successful check, configuration snapshot or revision reference, observed
  trigger state, and relevant activity IDs/cursors.
- Unresolved failures, uncertain writes, and findings already reported.

Start each check with current agent/account selection. Detect account changes
before combining new and old measurements. Refresh pending/running or otherwise
unresolved activity: an existing item can change status without becoming a new
item. Pagination cursors help traverse history; they are not a complete event
subscription. Honor rate limits and use bounded retries for transient read errors.

Notify on meaningful changes, completion of a tracked issue, failure, or required
user action. Report stale monitoring when reads keep failing; do not repeat the
last healthy state as current. Stay quiet on unchanged checks unless the user
requested periodic reports.

Keep diagnosis separate from intervention. Apply a repair only when the user has
authorized that management action, and verify it afterward. Requests for a trade,
order/protection change, or a manual rerun go to the DXAP website, including when
the user frames the action as a diagnostic test.
