# pcb.dog for agents

Let your agent quote and order printed circuit boards. It exports Gerbers from KiCad, pre-checks them for
manufacturing problems, compares USD prices and door-to-door delivery dates, and orders once you approve the summary.
You pay through a link; the agent never handles card details.

2, 4 and 6 layer FR-4 boards, 10 × 10 to 400 × 500 mm, HASL or ENIG. US and EU shipments are DDP (duties prepaid).
No API key or account.

## Claude Code

```
/plugin marketplace add pcb-dog/skills
/plugin install pcb-dog@pcb-dog
```

This adds the `order-pcb` skill and connects the pcb.dog MCP server. Then ask for what you need, for example
"get me 5 of this board, cheapest option", or run `/pcb-dog:order-pcb`.

## Any MCP client

Add the remote server `https://pcb.dog/mcp` (Streamable HTTP, no authentication). In Claude Code without the plugin:

```
claude mcp add --transport http pcb-dog https://pcb.dog/mcp
```

Tools: `get_pcb_quote`, `analyze_gerbers`, `find_cost_reductions`, `get_capabilities`, `create_order`,
`confirm_order`, `save_quote`. It is also in the official MCP Registry as `dog.pcb/mcp`.

## Other agents

[`skills/order-pcb/SKILL.md`](skills/order-pcb/SKILL.md) follows the [Agent Skills](https://agentskills.io) format:
copy the `order-pcb` folder into your agent's skills directory. Without MCP it uses the plain HTTP API.

## Links

- API and agent guide: https://pcb.dog/en/agents
- llms.txt: https://pcb.dog/llms.txt · OpenAPI: https://pcb.dog/openapi.json
- KiCad and Altium design rules: https://pcb.dog/en/rule-files
- Contact: hello@pcb.dog

MIT licensed.
