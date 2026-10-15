# Maintaining DXAP skills

Keep each skill usable as a standalone folder in any compatible harness.
Core instructions live in each `SKILL.md`; conditional detail belongs in that
skill's references. Use portable Agent Skills frontmatter. Add
harness-specific metadata only when it has a clear installation benefit.

## Contract changes

Check behavior against the current research, trading, and management tool
definitions and HTTP schema.
Update the relevant reference, the main skill if its decisions change, and
`metadata.version` together. Refresh the installation after changing the package.
If a deployed schema differs from the documented contract, resolve compatibility
before describing new functionality as available.

Check these contracts carefully:

- Action identity and exact replay across transports and key rotation.
- Account binding, paper/live mode, and independently selectable scopes.
- Receipt status versus execution, order, and fill evidence.
- Order revisions and complete replacement of position exits.
- Market filters, observation times, valuation completeness, and pagination.
- Research tool availability for each connection, time-window semantics, article
  attribution, X source relationships, and incomplete technical analysis.
- External-agent creation and account binding as prerequisites for `dxap-trading`.
- Hosted configuration section replacement, directive lifecycles, scope requirements,
  schedule timing, activity attribution, and recovery after uncertain writes.
- The manager's website-only boundary for hosted trades, trade proposals, order
  changes, and forced turns, including indirect workarounds through configuration.

Tool schemas are authoritative for exact fields. References explain semantics
and show focused examples; avoid maintaining a second full schema by hand.

## Validation

For each change:

1. Validate the skill's frontmatter against the
   [Agent Skills specification](https://agentskills.io/specification).
2. Check local Markdown links and parse JSON/TOML examples.
3. Read the changed instructions as a complete installed folder. References must
   resolve without the backend repository, maintainer filesystem, or chat history.
4. Exercise the relevant [offline scenarios](evals/scenarios.md) using synthetic
   observations and no trading connection. Evaluate the resulting decisions,
   not exact wording.

For discovery, check that market briefs select `dxap-research`, external-agent
portfolio/order tasks select `dxap-trading`, and hosted-agent monitoring,
troubleshooting, or configuration tasks select `dxap-agent-manager`. Resolve
ambiguous agent type before writes; hosted trades go to the website.
A research-only connection must remain sufficient for news, curated X summaries,
and technical analysis. Hosted management uses REST for all operations, including
listing, creating, and canceling directives. Settings creates general account REST keys;
external-agent MCP keys come from the agent's Connection panel. Do not recommend
MCP scopes for hosted supervision or a research-key creation option in Settings.

Connection verification should use reads. A paper execution test requires a
selected paper account and an authorized action. Keep account observations and
credentials outside the repository.

## Public content

Write instructions from the caller's perspective: what the tool does, when to
use it, required inputs, and meaningful results. Implementation relationships
such as "shared" tools, backend services, storage, caches, and source routing do
not belong in skill instructions. Explain differences between connection URLs
only when they change credentials or available capabilities.

Publish only user-facing contracts and synthetic examples. Exclude credentials,
real account/action IDs, local paths, private links, infrastructure details,
deployment secrets, internal test fixtures, and operational logs. Confirm endpoint
rollout before removing the availability note from the setup guides.

Installing the skill does not grant trading authority or schedule work. Preserve
the user's existing authorization and strategy rather than adding unconditional
approval prompts or prescribing a trading strategy.
Hosted autonomous schedules remain on DXAP; the supervisor's monitoring schedule
belongs in the harness. Offline manager checks must not create trades or turns.
