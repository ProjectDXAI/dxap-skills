# Repository guidance

This repository distributes public instructions for DXAP research, external-agent
trading, and hosted-agent management.

- Keep each folder under `skills/` independently installable. Links from a skill
  must resolve inside that folder or point to public documentation.
- Write for the agent using DXAP. Describe what a tool does, when to use it,
  required inputs, and useful results. Do not expose backend architecture or
  label capabilities as "shared," "internal," or "reused." Describe supported
  connections and permissions only when they affect the user's task.
- Use the current research, trading, and management contracts. Keep
  receipt recovery, account binding, scope requirements, and exit replacement
  semantics explicit when editing related instructions.
- For external agents, keep strategy and scheduling in the user's harness.
  For hosted agents, DXAP owns autonomous strategy and execution schedules;
  the local harness owns supervision and its monitoring cadence. Preserve
  existing user authorization; installation grants no permission to trade or
  start monitoring.
- `dxap-trading` requires an external agent created on DXAP and a credential bound
  to that agent and account. It cannot trade for hosted agents.
- `dxap-agent-manager` monitors, troubleshoots, and configures hosted agents.
  Hosted trade requests, trade proposals, order/protection mutations, and forced
  turns belong on the DXAP website. Do not route around that boundary through
  chat, directives, triggers, or schedule/status edits.
- Keep connection setup distinct: hosted-agent management uses account REST and
  keys from Settings → API keys; external agents use MCP and keys
  from their own Connection panel. Do not recommend MCP setup for hosted management.
- Research reuses the external MCP connection with market-data permission. Existing
  research-only MCP keys remain usable without a trading identity; do not advertise
  a research-key creation option in Settings or mix account and external scopes.
- Include only public product concepts and synthetic examples. Never copy keys,
  account observations, private URLs, operational fixtures, or local paths into
  the package.
- Validate frontmatter, links, and structured examples after documentation edits.
  Use `evals/scenarios.md` for meaningful behavioral checks.
- Repository work does not authorize connecting to a trading account or
  submitting an order. Test documentation offline unless the user explicitly
  requests an account operation.
