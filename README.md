# OneUp Today agent skills

[![skills.sh](https://skills.sh/b/oneup-today/skills)](https://skills.sh/oneup-today/skills)

Agent skills for [OneUp Today](https://oneup.today), the AI agents that find people on Reddit and X who are already asking for what you sell, and draft the first message to each.

Install into Claude Code, Codex, Cursor, OpenCode and other agents that support skills:

```bash
npx skills add oneup-today/skills
```

## Skills

| Skill | What it does |
|---|---|
| [`oneup-today`](skills/oneup-today/SKILL.md) | Drive OneUp Today from the terminal through the [`@oneup_today/cli`](https://www.npmjs.com/package/@oneup_today/cli): find leads, read replies, draft and send DMs, approve public replies, publish and schedule posts across X, Reddit, LinkedIn, Instagram, Threads, Facebook and YouTube, turn commits into build-in-public posts, run the SEO agent, and install the Front Desk website chatbot. |

## Requirements

- Node.js 18+ (the skill runs `npx @oneup_today/cli`)
- A OneUp Today account: sign up free at [oneup.today](https://oneup.today), then run `npx @oneup_today/cli login`

Prefer tool calls over a shell? The same backend is available as an MCP server: [oneup.today/mcp](https://oneup.today/mcp).

## License

MIT
