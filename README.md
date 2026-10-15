# DXAP Agent Skills

Connect your own agent harness to DXAP for market research, paper or live
perpetual trading through an external agent, and hosted-agent supervision.
The skills follow the [Agent Skills standard](https://agentskills.io/specification) and work with
Codex, Claude Code, or a custom harness that loads Markdown instructions.

| Skill | Use it for | Connection and key setup |
| --- | --- | --- |
| [dxap-agent-manager](skills/dxap-agent-manager/SKILL.md) | Monitor, troubleshoot, and configure hosted agents, including directives and schedules | REST; select account permissions on a key created in Settings → API keys |
| [dxap-trading](skills/dxap-trading/SKILL.md) | Portfolio review, authorized trades, order management, and receipt recovery for an external agent created on DXAP | MCP; create a bound key on that agent's Connection panel |
| [dxap-research](skills/dxap-research/SKILL.md) | News, article follow-up, curated X commentary, and technical analysis | Research tools on the external agent's MCP connection, or an existing research-only MCP connection |

## Connect

**Hosted-agent management uses REST.** Create a key in **Settings → API keys**
and select the account read/write permissions needed for the task. These are
general account API keys for any API client or integration, not agent-bound keys
or assistant-specific credentials. The manager skill uses their account REST
permissions and retains the user's target-agent boundaries. Monitoring needs
read permissions; configuration changes need the relevant write permissions.
No MCP connection or MCP permission is required for this workflow.
Install `dxap-agent-manager` and start with: “Inspect my hosted agent's settings,
schedule, and recent activity. Explain any issues without changing anything.”
See [manager setup](skills/dxap-agent-manager/references/setup.md).

**External agents connect through MCP.** Create an external agent on DXAP with a
selected paper or live account, then use its **Connection → Connect agent** panel.
The key is bound to that agent and account. Create and manage these keys on the
agent page, not in Settings. Install `dxap-trading` and start with: “Show my account
mode, portfolio, trading limits, and recent actions. Do not submit orders.”
See [trading setup](skills/dxap-trading/references/setup.md).

For market research, reuse the external connection with market-data permission
and install `dxap-research`. Start with: “Summarize BTC news and curated X
commentary from the past day, with sources.” Existing research-only MCP keys
remain supported; see [research setup](skills/dxap-research/references/setup.md).

| Workflow | Address |
| --- | --- |
| Hosted management REST, including all directive operations | `https://api.dxap.ai/api/v1/account` |
| External-agent MCP | `https://mcp.dxap.ai/mcp` |
| Optional external-agent REST for custom clients and receipt recovery | `https://api.dxap.ai/api/v1/account/external-agent` |
| REST schema | `https://api.dxap.ai/openapi.json` |

Hosted-agent trade requests and manually forced turns belong on the DXAP website.
The manager skill does not create trade proposals, submit orders, or force turns.
External-agent trading requires an external agent created on DXAP, its bound key,
and the user's trading authorization. Installing a skill grants neither access
nor trading permission.

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
- [Manager setup](skills/dxap-agent-manager/references/setup.md): REST access, account permissions, and directive routes.
- [Management API](skills/dxap-agent-manager/references/api.md): configuration and monitoring routes.
- [Settings and directives](skills/dxap-agent-manager/references/management.md): safe updates, lifecycles, schedules, and recovery.
- [Monitoring](skills/dxap-agent-manager/references/monitoring.md): diagnosis, evidence, and recurring supervision.
- [Maintainer guidance](CONTRIBUTING.md): compatibility checks and public content conventions.
- [Behavioral scenarios](evals/scenarios.md): offline checks for all three skills.

## License

[MIT](LICENSE).
