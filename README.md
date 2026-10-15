# DXAP Agent Skills

Connect your own agent harness to DXAP for market research, paper or live
perpetual trading through an external agent, and hosted-agent supervision.
The skills follow the [Agent Skills standard](https://agentskills.io/specification) and work with
Codex, Claude Code, or a custom harness that loads Markdown instructions.

| Skill | Use it for | Account access |
| --- | --- | --- |
| [dxap-research](skills/dxap-research/SKILL.md) | News, article follow-up, curated X commentary, and technical analysis | Public MCP API key; no trading account required |
| [dxap-trading](skills/dxap-trading/SKILL.md) | Portfolio review, authorized trades, order management, and receipt recovery for an external agent created on DXAP | API key bound to that external agent and its paper or live account |
| [dxap-agent-manager](skills/dxap-agent-manager/SKILL.md) | Monitor, troubleshoot, and configure hosted agents, including settings, directives, and schedules | User API key with the required account read/write scopes; REST access |

`dxap-trading` requires an external agent created on DXAP. Hosted-agent trade
requests and manually forced turns belong on the DXAP website; the manager skill
does not create trade proposals, submit orders, or force turns.

## Connect

For research, create a DXAP API key with `use:mcp:public`, configure the research
connection, and install `dxap-research`. Start with: “Use DXAP research to summarize
BTC news and curated X commentary from the past day, with sources.” See
[research setup](skills/dxap-research/references/setup.md).

For trading, create an external agent on DXAP with a selected paper or live
account and issue a key bound to both. Configure the trading connection and
install `dxap-trading`. Start with: “Show my account mode, portfolio, trading
limits, and recent actions. Do not submit orders.” See
[trading setup](skills/dxap-trading/references/setup.md).

For hosted-agent management, configure account REST access with the read scopes
needed for monitoring. Add agent/trigger write scopes for authorized changes.
Install `dxap-agent-manager` and start with: “Inspect my hosted agent's settings,
schedule, and recent activity. Explain any issues without changing anything.”
See [manager setup](skills/dxap-agent-manager/references/setup.md). MCP is optional
for directive operations; the complete manager workflow uses REST.

| Interface | Address |
| --- | --- |
| MCP (research, account directives, or external trading by credential) | `https://mcp.dxap.ai/mcp` |
| Trading REST | `https://api.dxap.ai/api/v1/account/external-agent` |
| Hosted management REST | `https://api.dxap.ai/api/v1/account` |
| REST schema | `https://api.dxap.ai/openapi.json` |

All MCP clients use the same URL. Account keys use `use:mcp:public`; keys bound
to external agents use `use:mcp:external`. Additional scopes determine the tools
available to each credential. The two transport scopes are not interchangeable.

Use `https://mcp.dxap.ai/mcp` for production unless explicitly configuring a
different DXAP environment.

## Install a skill

Copy the entire chosen skill folder, including its references and license.
Replace `<skill-name>` with `dxap-research`, `dxap-trading`, or `dxap-agent-manager`:

| Harness | Personal installation | Workspace installation |
| --- | --- | --- |
| Codex | `~/.agents/skills/<skill-name>/` | `.agents/skills/<skill-name>/` |
| Claude Code | `~/.claude/skills/<skill-name>/` | `.claude/skills/<skill-name>/` |

For example, for a first personal Codex installation, run from this checkout:

```sh
(
  skill=dxap-research
  destination="$HOME/.agents/skills/$skill"
  if [ -e "$destination" ] || [ -L "$destination" ]; then
    echo "A skill already exists at $destination; review it before updating." >&2
    exit 1
  fi
  mkdir -p "$(dirname "$destination")"
  cp -R "skills/$skill" "$destination"
)
```

Set `skill=dxap-trading` for trading or `skill=dxap-agent-manager` for hosted-agent
management. For Claude Code, use `$HOME/.claude/skills/$skill` as the destination.
For updates, review the changes and replace the installed folder with the new
version, keeping the entrypoint and references together.

In Codex CLI or the IDE extension, invoke `$dxap-research`, `$dxap-trading`, or
`$dxap-agent-manager`. In Claude Code, invoke `/dxap-research`, `/dxap-trading`, or
`/dxap-agent-manager`. Both can also select a skill from its description.
Configure the needed MCP connection or REST access separately.
See the official [Codex skill documentation](https://learn.chatgpt.com/docs/build-skills)
and [Claude Code skill documentation](https://code.claude.com/docs/en/skills).

For a custom harness, load the chosen skill's `SKILL.md` and make its references
available. Connect an MCP client or expose the relevant REST operations.
For external trading, keep strategy, authorization, and scheduling in the harness.
For hosted management, DXAP runs the agent's autonomous strategy and execution
schedule; the harness keeps supervision instructions, monitoring cadence, and
continuity. Installing a skill does not start a recurring watch.

## Contents

- [Research skill](skills/dxap-research/SKILL.md): select evidence and produce sourced market briefs.
- [Research setup](skills/dxap-research/references/setup.md): credentials and MCP configuration.
- [Research tool guidance](skills/dxap-research/references/tools.md): news, X summaries, and technical analysis.
- [Trading skill](skills/dxap-trading/SKILL.md): context, actions, and recovery.
- [Trading setup](skills/dxap-trading/references/setup.md): accounts, credentials, and MCP configuration.
- [REST API](skills/dxap-trading/references/rest-api.md): route mapping, examples, and pagination.
- [Order lifecycle](skills/dxap-trading/references/order-lifecycle.md): uncertain outcomes, revisions, and protection.
- [Agent manager](skills/dxap-agent-manager/SKILL.md): hosted supervision and authorized management.
- [Manager setup](skills/dxap-agent-manager/references/setup.md): REST access, scopes, and optional MCP directives.
- [Management API](skills/dxap-agent-manager/references/api.md): configuration and monitoring routes.
- [Settings and directives](skills/dxap-agent-manager/references/management.md): safe updates, lifecycles, schedules, and recovery.
- [Monitoring](skills/dxap-agent-manager/references/monitoring.md): diagnosis, evidence, and recurring supervision.
- [Maintainer guidance](CONTRIBUTING.md): compatibility checks and public content conventions.
- [Behavioral scenarios](evals/scenarios.md): offline checks for all three skills.

## Replacing the earlier skill

This is the successor to `dxap-skill` for externally managed trading. The earlier
`dxap-hyperagent-api` skill covered hosted agent management, chat, and triggers.
Use `dxap-trading` for an identity with `runtime_mode: external`. Remove the old
skill from that trading workspace if it is no longer needed. Existing broad
account keys must be replaced with credentials bound to the external agent and
its account; installing this skill does not change a key's scopes.

Use `dxap-agent-manager` for hosted monitoring, troubleshooting, and settings
management. Its scope excludes trade creation and manually forced turns, even
when an older skill or a configured credential exposes those operations.

## License

[MIT](LICENSE).
