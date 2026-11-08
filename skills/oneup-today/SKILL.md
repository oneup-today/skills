---
name: oneup-today
description: Find customers on Reddit and X and run AI outreach with OneUp Today from the terminal. Covers finding leads, reading the pipeline, drafting and sending DMs, approving public replies, publishing and scheduling posts on X, Reddit, LinkedIn, Instagram, Threads, Facebook and YouTube, turning git commits into build-in-public posts, the SEO article agent, and the Front Desk website chatbot. Use it when the user mentions OneUp Today, or asks to "find people on Reddit who need my product", "any new replies?", "who are my best leads", "draft a DM to this lead", "post this to X", "schedule a tweet", "turn today's commits into a post", or "add a chatbot to my site".
---

# OneUp Today

[OneUp Today](https://oneup.today) finds people on Reddit and X who are already asking for what the user sells, drafts the first message to each, and runs the follow-ups. This skill drives it through the official CLI, [`@oneup_today/cli`](https://www.npmjs.com/package/@oneup_today/cli), which works from any agent that can run a shell command.

## Setup

```bash
npx @oneup_today/cli whoami --json      # no install needed
npm install -g @oneup_today/cli          # or install once, then use `oneup`
```

- Not signed in? Ask the user to run `oneup login` (it opens the browser and stores the key at `~/.config/oneup/config.json`). In CI, set `ONEUP_API_KEY`. Never print, log or echo the key.
- No account yet? Sign-up is free at [oneup.today](https://oneup.today) and comes with starting credit.
- Commands below are written as `oneup ...`. Without a global install, prefix them with `npx @oneup_today/cli`.

## Rules

1. Add `--json` to every command and parse that. Without it you get a human table.
2. Reads are free and safe. Before any command that writes, sends, posts or costs money, tell the user exactly what will happen and get a yes.
3. On failure, read the `error` field and relay it. Do not blind-retry.
4. `oneup <command> --help` has the exact flags. `oneup tools` prints the live catalog if this file looks out of date.
5. Data is scoped to the signed-in account. You only ever see the user's own agents, leads and posts.

## Common tasks

### First run: find leads for a product
```bash
oneup campaigns list --json            # empty list means a new account
oneup find-leads https://their-site.com --json
```
`find-leads` adds the product, creates a draft-only Reddit agent and runs its first search. Nothing is sent, and it costs one agent run, so confirm first. The search takes a few minutes; check `oneup leads list --json` afterwards.

### "Any new replies?" / "What needs my approval?"
```bash
oneup leads stats --json                          # pipeline totals, start here
oneup leads by-status engaged --json              # engaged = they replied
oneup conversations list <campaign_id> --filter unread --json
oneup inbox search "<username or text>" --json    # when the user names a person
oneup replies list --json                         # public reply drafts awaiting approval
```

### Draft and send a DM
```bash
oneup readiness --json                                     # can real Reddit sends go out now?
oneup conversations get <campaign_id> <conversation_id> --json
oneup drafts send <campaign_id> <conversation_id> --json   # sends the stored draft
```
If readiness says not ready, sends draft only until the user opens the OneUp Today Chrome extension signed in to Reddit. Say so instead of retrying. Show the exact text and confirm before any send.

### Publish or schedule a post
```bash
oneup post --platform x --text "..." --json
oneup post --platform reddit --title "..." --subreddit SideProject --text "..." --json
oneup post --platform x --text "..." --at "2026-12-01 09:00" --tz Asia/Kolkata --json
oneup compose --platforms x,linkedin,instagram,youtube --text "..." --media ./demo.mp4 --title "..." --json
```
`oneup post` QUEUES the post: it goes out through the user's OneUp Today browser extension the next time their browser is open and signed in to that platform. Tell the user it is queued, not live. `oneup compose` saves one multi-platform draft and returns a `review_url`; nothing is published.

### Build in public from the repo
```bash
oneup post from-commits --dry-run --json                     # today's local git log -> draft copy
oneup post from-changes --message "shipped dark mode" --media ./after.png --json
```
Use `--dry-run` first when the user wants to read the wording before it is queued.

### Put a chatbot on the user's website
```bash
oneup frontdesk install
```
Creates the Front Desk chatbot if needed, builds its knowledge base from the user's site and writes the script tag into their HTML entry point. Safe to re-run. `--print` only prints the snippet.

### SEO articles
`oneup seo sites`, `oneup seo pending`, then `oneup seo approve <site_id> <article_id>` (publishes, confirm first). `oneup seo write <site_id>` generates one article and costs one article.

## Billing and errors

Actions that make something (drafts, posts, articles) debit the user's prepaid wallet; reads are free and a failed action is refunded.
- "Your wallet balance is too low": stop and tell the user to top up at the URL in the error. A retry fails the same way.
- "Rate limit exceeded" / 429: wait the number of seconds named, retry once.
- "Invalid or expired API key": the user runs `oneup login`.

## More

- Full command reference with every flag and output shape: [references/cli-reference.md](references/cli-reference.md)
- Prefer tool calls over a shell? The same backend is an MCP server: see [oneup.today/mcp](https://oneup.today/mcp).
- Help docs from the terminal: `oneup docs "<question>" --json`
