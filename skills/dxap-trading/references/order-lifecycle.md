# Order lifecycle and recovery

Read this guide before order management, protection changes, or uncertain-result
recovery. Use current tool schemas for argument details.

## Action identity and continuity

A new action gets a fresh UUID `client_action_id`. Save it with the bound account,
operation, and exact command before submission. Identical retries use that same
ID, even across MCP and REST or after key rotation. Changing inputs under the
same ID causes a conflict, including when the original action was rejected.

After a confirmed rejection or failure, a revised decision is a new action with
a new ID. Reconsider the cause first. Never use a new ID to retry an unresolved
action or bypass a policy rejection.

Maintain a private action journal in the harness, outside this skill package.
It should preserve pending IDs and exact commands across restarts. On resume,
reconcile those IDs with DXAP before making another trading decision. If the
journal is unavailable, review recent activity and paginate as needed; an absent
action in the first page is not proof that a submission failed.

## Read a receipt

Use `get_action_receipt` with `client_action_id` or REST
`GET /requests/{clientActionID}`. The receipt's `status` tracks the action;
`execution.status`, orders, and confirmed fill evidence describe the trading
outcome.

| Receipt status | Interpretation and next step |
| --- | --- |
| `submitting` | Outcome unresolved. Retrieve the receipt; pause new mutations. |
| `reconciling` | Outcome unresolved. Continue bounded receipt reads; pause new mutations. |
| `recorded` | Execution outcome available. Inspect it for rejection, resting orders, partial fills, or fills. |
| `rejected` | Action not admitted. Inspect the reported cause before a new decision. |

An accepted order is not a confirmed fill. A market fill may be confirmed before
its DXAP order UUID is available. Refresh the receipt to obtain
`execution.order_id` or `execution.orders[].id` for management. Venue identifiers
such as `provider_order_id` are not the DXAP UUID accepted by management tools.

## Recover after a timeout or disconnect

1. Keep the original action ID and exact command. A transport error alone does
   not establish whether the action executed.
2. Read that receipt. Use recent activity if needed to identify the action.
3. While the result is unresolved, make no new mutations. Poll with bounded
   backoff, respecting server retry guidance and the harness's task deadline.
   As a harness convention, a foreground task can stop polling after one minute
   and report the unresolved ID; this is not a DXAP resolution guarantee.
4. If the task's wait budget expires, persist the unresolved state and report
   it. Resume receipt checks on a later authorized run. Do not schedule new work
   unless the user's harness/task already calls for it.
5. Once resolved, inspect execution and current orders, then refresh relevant
   account context before another decision.

A missing receipt, failed read, generic tool error, or lost credential is not
proof that nothing executed. Retain uncertainty and restore authorized read
access or seek DXAP support. If replay is appropriate after an admission failure,
send the original command with its original ID; never reconstruct it with changed
arguments. DXAP may continue resolving an accepted action while the harness is
offline, trading is disabled, or its key has been revoked. Resolution has no
guaranteed maximum duration.

## Edit or cancel an order

Fetch `get_order` using a DXAP order UUID from a receipt or portfolio. Read
`revision`, `can_edit`, `can_cancel`, remaining quantity, and observation time.
Copy `order_id` and `expected_revision` from the same latest result.

- To modify, supply `limit_price` or `remaining_size`, or both. Remaining size is
  the unfilled quantity in asset units, represented as a decimal string.
- A revision conflict requires a fresh order read and reconsideration. If the
  revised action is still intended and the original is resolved, give it a new
  action ID. Do not change the revision under the old action ID.
- After modification, use the returned order ID and revision for later actions.
- Partially filled entries with attached exits must be canceled instead of
  edited. Canceling an entry removes its waiting attached exits; confirmed fills
  remain in the position. Review protection for that remaining position.
- Canceling one eligible take-profit or stop-loss order preserves its parent
  and siblings.

## Entry exits and position protection

Market entry take-profit/stop-loss orders cover the confirmed fill, including
partial fills. Entry execution can succeed even if exit placement fails. Report
both outcomes and review position protection before describing the position as
protected.

Limit entry take-profit/stop-loss orders activate only after the entire entry
fills. A partial limit entry is not yet protected by those attached exits.
An edit retains attached exit prices and resizes them with the entry when the
order is eligible for editing.

`set_perp_exit_orders` replaces the **complete DXAP-managed position protection**:

- Supply every take-profit and stop-loss that should remain.
- Up to four take-profit targets are supported. Each `trigger_price` and
  `close_percent` is a decimal string. Percentages refer to the current position
  and must total at most 100. The stop covers the remaining position.
- `take_profits: []` explicitly removes take-profits. Omission or `null` has the
  same effect; a supplied stop-loss is independent of that list.
- An omitted stop-loss removes the position stop. A request to change just the
  stop price therefore still needs all take-profits that should be retained.

Position exit replacement/cancellation leaves entry orders and their waiting
attached exits unchanged. `cancel_perp_exit_orders` cancels DXAP-managed position
exits for the symbol and leaves orders placed outside DXAP in place.

Pausing an agent or revoking its key does not cancel accepted orders. Those
orders may fill. Cancellation requires its own authorized action and verification.
