# DXAP Agent Skills

Skills for market research, external-agent trading, and hosted-agent management
with Codex, Claude Code, or another compatible harness.

## Choose a skill

| Skill | Purpose | Setup guide |
| --- | --- | --- |
| [dxap-agent-manager](skills/dxap-agent-manager/SKILL.md) | Monitor, troubleshoot, and configure hosted agents through REST | [Account API access](skills/dxap-agent-manager/references/setup.md) |
| [dxap-trading](skills/dxap-trading/SKILL.md) | Review portfolios, execute authorized trades, and manage orders through an external agent created on DXAP | [External-agent connection](skills/dxap-trading/references/setup.md) |
| [dxap-research](skills/dxap-research/SKILL.md) | Research news, curated X commentary, and technical analysis through MCP | [Research access](skills/dxap-research/references/setup.md) |

## Install

Copy the entire folder from `skills/<skill-name>/`, including its references and
license, into one of these locations:

| Harness | Personal installation | Project installation |
| --- | --- | --- |
| Codex | `~/.agents/skills/<skill-name>/` | `.agents/skills/<skill-name>/` |
| Claude Code | `~/.claude/skills/<skill-name>/` | `.claude/skills/<skill-name>/` |

Use a skill name from the table above. When updating, replace the installed
folder with the complete new version.

Invoke `$<skill-name>` in Codex or `/<skill-name>` in Claude Code, followed by your
request. For a custom harness, load the skill's `SKILL.md` and make its references
available.

## Configure access

**Hosted-agent management:** Create a key in **Settings → API keys** and select
the account REST permissions needed for your task. These keys are account-scoped,
not bound to an individual agent. Use `https://api.dxap.ai/api/v1/account` for
monitoring, configuration, schedules, and directives.

**External-agent trading:** Create an external agent on DXAP with a selected
paper or live account. Open its **Connection → Connect agent** panel to create a
key bound to that agent and account. Configure your MCP client with
`https://mcp.dxap.ai/mcp` using the setup guide above.

**Research:** Reuse the external agent's MCP connection with market-data
permission, or use an existing research-only MCP connection.

Keep API keys in your client's secret configuration. Installing a skill does not
configure API access or authorize trading. Hosted trade requests and manually
forced turns belong on the DXAP website.

## License

[MIT](LICENSE).
