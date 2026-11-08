# OneUp Today CLI: full command reference

`oneup` drives OneUp Today (AI Reddit/X outreach) from the shell. If you are an AI agent
(Cursor, Claude Code, Codex, or a shell script), you can run these commands directly.

Run install-free with `npx @oneup_today/cli <command>`, or `oneup <command>` if installed globally.

## Golden rules (read first)
1. Add `--json` to EVERY command and parse that. Without it you get a human table, not JSON.
2. Auth is already set up by the user (`oneup login`, stored at `~/.config/oneup/config.json`) or via
   the `ONEUP_API_KEY` env var. Never print, log, or echo the key.
3. All data is scoped server-side to the user's own account (and company). You cannot see or touch
   anyone else's campaigns, leads, or posts.
4. Before any WRITE or SEND command, tell the user exactly what you are about to do and get
   confirmation. Reads are safe; writes/sends are not.
5. On failure, the JSON has an `error` (or the process exits non-zero). Read it and relay it to the
   user. Do not blind-retry the same call.
6. `oneup <command> --help` shows exact flags. `oneup tools` prints the live backend tool catalog if
   anything here looks out of date.

## Reads (safe, no side effects)
- `oneup whoami` - which account you are acting as.
- `oneup campaigns list [--platform reddit|x] [--status scheduled|running] [--limit N]`
- `oneup campaigns get <campaign_id>`
- `oneup campaigns activity <campaign_id>` - audit log for a campaign.
- `oneup runs list <campaign_id>` / `oneup runs get <campaign_id> <run_id>`
- `oneup leads list [--campaign <id>] [--status new|engaged|qualified|trail|paid]` / `oneup leads get <lead_id>`
- `oneup leads stats` - pipeline totals by status, source and agent. Start here for "how are my leads doing".
- `oneup leads by-status <new|engaged|qualified|trail|paid|disqualified>` - engaged = they replied, paid = converted.
- `oneup leads explain <lead_id>` - why that lead got its 0-100 score.
- `oneup leads reasons` - the valid disqualification reasons for `oneup leads status`.
- `oneup posts list [--status draft|published|...] [--platform x] [--origin agent|composer]` - see "Your posts" below.
- `oneup posts get <post_id>` - one post in full, with where it published.
- `oneup frontdesk install` - put the Front Desk chat widget on the user's website in one command.
- `oneup frontdesk status` / `oneup frontdesk list [--unread] [--escalated]` / `oneup frontdesk get <conversation_id>` - see "Front Desk" below.
- `oneup log list [--consumed false]` - your work-log notes; unused ones feed a `--source log` schedule.
- `oneup conversations list <campaign_id> [--filter unread|draft|replied]`
- `oneup conversations get <campaign_id> <conversation_id>`
- `oneup socials list [--platform reddit|x]` - connected accounts and their `social_id`s.
- `oneup products list` - you need a `product_id` to create a campaign.
- `oneup docs "<question>" [--page /connections]` - search OneUp Today's own help docs. Use it for "how does OneUp Today do X" or "what does this setting mean", not for the user's data.
- `oneup readiness [--platform reddit] [--social <id>]` - see "Readiness" below.

## Writes (mutate or send - confirm with the user first)
- `oneup find-leads <website_url> [--product <id>] [--social <id>]` - the first-run path for a new account or a new product: adds the product, creates a draft-only Reddit agent and runs its first search. Nothing is sent. It costs one agent run, so confirm first. Discovery takes a few minutes; check `oneup leads list` after.
- `oneup products add <website_url>` - add a product without starting a search (the same URL twice returns the existing one).
- `oneup campaigns create --product <id> --name "..." [--platform reddit|x] [--subreddits a,b] [--search-terms k1,k2] [--theme "..."] [--frequency once|daily|weekly] [--messaging auto_send|draft_for_review]`
- `oneup campaigns update <campaign_id> [--theme "..."] [--target 5] [--set key=value]`
- `oneup campaigns run|pause|resume <campaign_id>`
- `oneup leads status <lead_id> <new|engaged|qualified|trail|paid|disqualified> [--reason "..."]`
- `oneup dm draft --post-text "..." [--subreddit x] [--product <id>]` - returns DM copy only; does NOT send.
- `oneup drafts send <campaign_id> <conversation_id> [--dm "..."] [--reply "..."]` - sends a message.
- `oneup post ...` - publish a standalone post. See "Publishing posts" (it is QUEUED, not instant).
- `oneup compose --platforms x,threads,instagram,facebook,linkedin,youtube ...` - save ONE draft across platforms, with a video. Nothing is published. See "Multi-platform drafts".
- `oneup post ... --at "..."` / `oneup schedule create ...` / `oneup log "..."` - schedule posts. See "Scheduling posts".
- `oneup search-terms add|remove "<term>"` - manage tracked Reddit search terms (every Reddit messaging agent searches them).
- `oneup leads note <lead_id> "<note>"` - private note on a lead. Never sent to anyone.
- `oneup inbox regenerate <agent_id> <conversation_id> [--instruction "shorter"]` - rewrite an unsent draft. COSTS one DM draft.
- `oneup posts delete <post_id>` - PERMANENT, and only for unpublished drafts. Confirm first.
- `oneup frontdesk reply <conversation_id> --message "..."` - appears LIVE on the user's website. Confirm first.
- `oneup seo update <site_id> [--quota N] [--set key=value]` / `oneup seo delete <site_id>` - delete is permanent.

## Publishing posts (`oneup post`) - important, the behavior is not obvious
Queues a standalone post to X or Reddit. It does **NOT** publish immediately. OneUp Today publishes
through the user's browser extension, so this command QUEUES the post; the user's OneUp Today extension
submits it the next time their browser is open and signed in to that platform.

X - a single tweet to the user's own timeline:
```
oneup post --platform x --text "your tweet text"
```

Reddit - a self-post. BOTH `--title` and `--subreddit` are REQUIRED (posting to a profile is not
supported):
```
oneup post --platform reddit --title "Post title" --subreddit somesubreddit --text "post body"
```

Optional on either: `--social <id>` to choose a specific connected account (from `oneup socials list`),
and `--media <fileOrKey>` (repeatable, max 4) to attach images or a short video. `--media` accepts a
LOCAL file path (jpg/png/gif/webp/mp4/mov, up to 250MB) uploaded to the user's OneUp Today media library for
you, or an existing media-library `s3_key`. On X the attachments are added to the post for you; on Reddit they show on the draft card for the user to attach manually.

Upload separately with `oneup upload <file>` (prints the `s3_key` to reuse across posts).

## Multi-platform drafts (`oneup compose`)
`oneup post` only knows X and Reddit. For Threads, Instagram, Facebook, LinkedIn, YouTube or TikTok, or
ONE draft that goes to several at once, use `oneup compose`. It saves the same draft the app composer's
Save Draft saves, so the user opens the printed `review_url`, checks every account's preview and posts.
NOTHING is published and nothing is charged: tell the user it is a draft.
```
oneup compose --platforms x,threads,instagram,facebook,linkedin,youtube \
  --text "default copy" --caption x="short X copy" --caption linkedin=@linkedin.txt \
  --media ./video.mp4 --cover ./cover.jpg \
  --title "YouTube title" --tags "startup,founders" --youtube-type short
```
- `--platforms` (REQUIRED): x, threads, instagram, facebook, linkedin, youtube, tiktok (Reddit stays on `oneup post`). Each needs a connected account (`oneup socials list`); with several on one platform add `--social <platform>=<social_id>`.
- Copy: `--text` is the default; `--caption <platform>=<text>` overrides it per platform. A value starting with `@` and naming a file is read from that file (multi-line copy).
- `--media` (repeatable, max 4): a local video/image path (uploaded for you) or an existing `s3_key`. `--cover`: image for the video cover (Instagram Reels, Facebook, YouTube).
- Rules the command checks for you: instagram, youtube and tiktok need media; youtube and tiktok need a video; youtube needs `--title` (max 100); Threads text max 500; X text over 280 returns a warning (Premium only).
- More: `--youtube-type short|video`, `--youtube-privacy public|unlisted|private`, `--instagram-type IMAGE|REELS|STORIES|CAROUSEL`, `--no-share-to-feed`, `--linkedin-visibility PUBLIC|CONNECTIONS`, `--product <id>`.

Returns (`--json`): `{ post_id, status: "draft", platforms, accounts, media_count, warnings, review_url }`. Read it back
with `oneup posts get <post_id>`; discard it with `oneup posts delete <post_id>` (confirm first).

## Reposts and quote posts (`oneup post repost` / `oneup post quote`)
Both act on an EXISTING post given by `--url` and queue for the extension like everything else.

Repost = a retweet on X, a crosspost on Reddit. No text of your own:
```
oneup post repost --platform x --url https://x.com/someuser/status/123456789
oneup post repost --platform reddit --url https://www.reddit.com/r/somesub/comments/abc123/... --title "Worth a read" --subreddit yoursub
```

Quote = your commentary plus the quoted post. On X it becomes a quote tweet; on Reddit it becomes a
self-post with your commentary and the link (Reddit has no native quote):
```
oneup post quote --platform x --url https://x.com/someuser/status/123456789 --text "this matches what we saw"
oneup post quote --platform reddit --url https://www.reddit.com/r/somesub/comments/abc123/... --text "my take" --title "Interesting finding" --subreddit yoursub
```

Rules: `--url` must be a full X status URL or Reddit post URL. Reddit always needs `--title` and
`--subreddit` (the destination). `repost` takes NO `--text` (use `quote` for commentary) and no
media. `quote` accepts `--media` like a regular post. X reposts and Reddit crossposts publish in
one click from the extension (no composer review step); X quotes open the prefilled composer for
the user to click Post.

## Continuing a SERIES on X (`oneup post followup`)
For a build-in-public series where each post quote-tweets the previous one (keeping the thread
chained and visible), use `followup`. It writes your new post as a QUOTE of your previous
PUBLISHED series post, resolving that post automatically, no `--url` needed:
```
oneup post followup --text "Day 2: shipped the quote-chain. Each update now links to the last."
oneup post followup --text "..." --media ./screenshot.png       # attach media
oneup post followup --quote-url https://x.com/you/status/123     # bootstrap: quote a specific post
```
X-only. The first `followup` (nothing published yet) queues a plain post to start the chain;
publish it from the extension and the next `followup` quotes it automatically. Optional
`--series <slug>` runs a named series; the default covers your single running one.

## Drafting posts FROM COMMITS (`oneup post from-commits`)
Build-in-public pipeline. Two modes:

LOCAL mode (recommended, the terminal-native path): run it INSIDE a git repo with NO repository
argument. It reads your local `git log` (no GitHub connection needed), writes plain founder-voice
posts, and queues drafts. Supports `--media` and scheduling (`--at`, see "Scheduling" below).
```
cd ~/code/my-app
oneup post from-commits                              # today's local commits -> a draft
oneup post from-commits --since-days 2 --media ./demo.png
oneup post from-commits --repo ../other-repo --dry-run
oneup post from-commits --at "tomorrow" --in 18h     # draft now, publish later
```

GitHub mode: pass `owner/name` (or a bare `name`) as the argument to fetch from the user's CONNECTED
GitHub account instead of local git. No `--media`/`--at` in this mode (use a recurring schedule for
GitHub-sourced posts, below).
```
oneup post from-commits my-repo --count 3 --since-days 7
oneup post from-commits owner/my-repo --platforms x,reddit --subreddit SideProject
```
Options: `--repo <path>` (local mode dir, default cwd), `--branch` (GitHub mode), `--count` (1-5;
default 3 GitHub, 1 local), `--platforms` (comma list, default x), `--subreddit` (with reddit),
`--social <id>`, `--since-days` (GitHub 7 / local 1), `--style <note>`, `--media` (local mode),
`--dry-run`. Use `--dry-run` first when the user wants to review before queuing.

## Drafting a post FROM YOUR CHANGES (`oneup post from-changes`)
When you just did some work and want to post about it: pass a short note on WHAT CHANGED and, optionally,
screenshots or a short video. The backend writes plain founder-voice copy FROM YOUR NOTE and attaches the
media to the draft (queued for the extension, same review flow). Unlike `from-commits`, this needs no
GitHub, just your description. The copy is written from the note; the media is attached, not analyzed.
```
oneup post from-changes --message "shipped inline cooldown timer fix; the Posts tab no longer freezes"
oneup post from-changes --message "new dark mode" --media ./before.png --media ./after.png
oneup post from-changes --message "..." --platforms x,reddit --subreddit SideProject --dry-run
```
Options: `--message <note>` (REQUIRED, drives the copy), `--media <fileOrKey>` (repeatable, max 4; a
LOCAL image/video path is uploaded for you, or pass an existing `s3_key`), `--platforms` (comma list,
default x), `--subreddit` (required with reddit), `--social <id>`, `--product <code>` (grounds the copy;
defaults to the user's first product), `--count` (1-5, default 1), `--style <note>`, `--dry-run`.
Use `--dry-run` first when the user wants to review the wording before it's queued.

Returns (`--json`):
```
{ "queued": 1, "media_count": 2, "posts": [ { "platform": "x", "text": "...", "based_on": "...",
  "post_id": "<id>", "status": "queued" } ] }
```

After queuing, tell the user:
- The post is QUEUED, not live. It publishes when their OneUp Today extension is open and signed in
  (on X it posts by itself; on Reddit they click Post on the prefilled page, then "Mark as posted").
- If `oneup socials list` shows no account for that platform, they must connect one at
  `app.oneup.today/connections` first.

Typical errors (returned in `error`): `text is required`, `title is required for Reddit posts`,
`subreddit is required for Reddit posts`, `No X account connected` / `No Reddit account connected`.

## Scheduling posts - one-off and recurring
Scheduling is free to set up; each post it publishes is charged. All of it still publishes through the user's browser extension: when a
post is due it is queued for the extension (the browser must be open at/after the scheduled time). Two shapes:

ONE-OFF: add `--at`/`--in` (+ optional `--tz`) to `oneup post`, `oneup post from-commits` (local
mode), or `oneup post from-changes`. `--at` takes `"YYYY-MM-DD HH:MM"` (interpreted in `--tz`, or
your machine's zone) or a full ISO time; `--in` is relative (`90m`, `2h`, `3d`). `--publish-mode`
is `auto` (default, publishes unattended) or `draft` (just surfaces it to review at that time).
```
oneup post --platform x --text "..." --at "2026-07-09 09:00" --tz Asia/Kolkata
oneup post from-changes --message "shipped X" --media ./shot.png --in 3h
```

RECURRING: `oneup schedule create` turns commits or a work log into a post on a cadence.
```
oneup schedule create --source commits --repos owner/app --at 18:00 --tz Europe/Berlin --days mon,tue,wed,thu,fri
oneup schedule create --source log --at 09:00 --tz Asia/Kolkata --series build-in-public --counter "Iteration - {n}/10000 of building X" --counter-start 47
```
Manage them: `oneup schedule list [--kind once|recurring]`, `oneup schedule get|pause|resume|delete <id>`,
`oneup schedule run <id> [--dry-run]` (fire now / preview), `oneup schedule reschedule <id> --at "..."`
(move a pending one-off). For the `log` source, add notes with `oneup log "what I shipped" [--media ...]`;
each recurring run folds in the notes you have not used yet.

Creating a schedule is FREE; the post it generates is charged when it publishes. If the wallet cannot
cover that, the create still succeeds and the response carries a `warning` -- surface it, do not treat
it as a failure. `--source commits` (recurring) uses the CONNECTED GitHub account; local-git
scheduling is the one-off `from-commits --at` path above.

## Replies (`oneup replies ...`) - the Reply Agent
The Reply agent writes PUBLIC comment replies. Reads are free; posting is public and irreversible.
- `oneup replies list [--platform reddit|x|instagram] [--limit N]` - drafts awaiting approval, all agents.
- `oneup replies history [--social <id>]` - what has already been posted.
- `oneup replies agent <agent_id> [--status draft|selected|queued|posted|discarded]` - one agent's replies.
- `oneup replies get <agent_id> <reply_id>` - one reply in full.
- `oneup replies select <agent_id> <reply_id>` - the agent often writes SEVERAL variants for one target
  (they share a group). Selecting one discards its siblings. This does NOT post it.
- `oneup replies post <agent_id> <reply_id> [--text "..."]` - PUBLISHES it. Always confirm with the user
  first and show them the exact text. `--text` replaces the draft before posting.
- `oneup replies cancel <agent_id> <reply_id>` - cancel a queued reply before the extension sends it.

Typical flow: `replies list --json` -> show the user the drafts -> they pick one -> `replies select`
-> confirm the final text -> `replies post`.

## Inbox (`oneup inbox ...`) - acting on a conversation
After `conversations list`, these act on one thread. All free except `followup`.
- `oneup inbox search "<query>" [--platform p] [--agent id]` - find a thread by username or message text.
  Use this FIRST when the user names a person instead of giving you a conversation_id.
- `oneup inbox read|unread <agent_id> <conversation_id>`, `oneup inbox star <a> <c> [--starred false]`.
- `oneup inbox reply <agent_id> <conversation_id> --message "..."` - SENDS a DM reply to that person.
  Confirm the exact text with the user first. Returns a job id; the send is async.
- `oneup inbox followup <agent_id> <conversation_id> [--regenerate]` - the objection-handling follow-up
  draft. COSTS one follow-up draft. Without `--regenerate` an existing current draft is returned free.
- `oneup inbox discard-draft <a> <c>` - bin an unsent draft. `oneup inbox cancel <a> <c>` - stop a queued DM.
- `oneup inbox discard-message <a> <message_id>` / `delete-message <a> <message_id>` - per-message cleanup.
  `delete-message` removes it from OneUp Today only; it does NOT unsend anything on the platform.

## Templates (`oneup templates ...`)
Saved message snippets. `list`, `create --name --content`, `update <id> [--name] [--content]`,
`delete <id>`, `use <id>` (records usage so ordering stays useful). All free.

## SEO Agent (`oneup seo ...`)
Writes and publishes SEO articles for a site on a cadence. Reads are free; writing an article costs
one article ($0.99). Rank tracking is billed MONTHLY per site, so reading ranks is free.
- `oneup seo sites` / `oneup seo get <site_id>` - the sites and their settings + usage.
- `oneup seo pending [--limit N]` - articles waiting on the user to approve. Show them BEFORE approving.
- `oneup seo approve <site_id> <article_id> [--at <iso>]` - PUBLISHES it (schedule with --at). Confirm first.
- `oneup seo unschedule <site_id> <article_id>` - pull a scheduled article back to review.
- `oneup seo write <site_id>` - generate one article now. COSTS one article. Runs in the background.
- `oneup seo plan <site_id>` / `oneup seo build-plan <site_id>` / `oneup seo add-keyword <site_id> "<kw>"`.
- `oneup seo improvement <site_id> <improvement_id> <apply|rewrite|reject|dismiss>`.
- `oneup seo traffic <site_id> [--article id] [--days N]` / `oneup seo ranks <site_id> <article_id>`.
- `oneup seo backlinks <site_id> [--action list|kit|check]` - kit WRITES submission copy.
- `oneup seo ask <site_id> "<question>"` - keyword-strategy Q&A, free.
- `oneup seo create --type <t> --name "..." --product <id>`, plus `pause`/`resume`/`verify`.

## Your posts (`oneup posts ...`) - the Content Agent's output

`oneup post` (singular) CREATES a post. `oneup posts` (plural) READS what already exists. Anything
`oneup post`, `post from-commits`, `post from-changes` or `post followup` queued shows up here.
- `oneup posts list` - newest first. `--origin agent` is what a tool or agent wrote; `--origin composer`
  is what the user wrote in the app. `--status draft` is written but not published.
- `oneup posts get <post_id>` - full body, media, and once live the URL plus likes and comments.
- `oneup posts delete <post_id>` - only works on a draft or suggested post. Deleting a PUBLISHED post
  would mean taking it down from X or Reddit, so the CLI refuses and points at the app.
- Recurring and one-off SCHEDULES are a different thing: use `oneup schedule list`.

## Front Desk (`oneup frontdesk ...`) - the Chatbot Agent

Front Desk is the chat widget on the user's OWN website. It answers visitors, captures leads, and
hands hot conversations over to the user.
- `oneup frontdesk install` - turn Front Desk ON and put the widget on their site, in one command.
  With nothing installed that is `npx @oneup_today/cli frontdesk install`; it signs the user in
  itself on first run, so this is genuinely the only command they need.
  Creates the chatbot if they have none, starts the knowledge-base build from their product and a
  crawl of their site, then writes the script tag into their HTML entry point (index.html,
  public/index.html) when there is one. For a framework it does NOT guess at a layout file: it names
  the stack, prints the tag, and prints a paragraph to paste into an AI assistant. Safe to re-run.
  `--print` prints the snippet and changes nothing; `--rebuild` re-crawls the site.
- `oneup frontdesk status` - is it live, what domain, unread count, knowledge-base size. Start here.
- `oneup frontdesk list [--unread] [--escalated]` - recent visitor conversations, newest first.
- `oneup frontdesk get <conversation_id>` - the full transcript plus the visitor's details.
- `oneup frontdesk read <conversation_id>` - clears it from the unread badge.
- `oneup frontdesk reply <conversation_id> --message "..."` - SENDS. It appears live in the widget on
  their site, and only works on a conversation that has been escalated to the user. Confirm first.

## Readiness (`oneup readiness`)
Real Reddit sends go out through the user's OneUp Today Chrome extension. If readiness reports NOT ready,
campaign runs and message sends will DRAFT only (nothing goes out) until the user opens the OneUp Today
Chrome extension signed in to Reddit. Surface that to the user instead of retrying blindly.

## Output & parsing
Every command takes `--json`. Lists return an object like `{"campaigns":[...],"count":N}` (the key
matches the noun: `campaigns`, `leads`, `conversations`, `socials`, `products`). Pipe into jq, e.g.
`oneup leads list --status qualified --json | jq '.leads[].username'`.

## Billing & limits (what to do when a command fails)
Reads are FREE and unmetered. Actions that make something (drafts, posts, articles) debit a prepaid
wallet at published per-action prices, and a failed action is refunded.

- **"Your wallet balance is too low"**: the error names what was needed, what is available, and a
  top-up URL. STOP and tell the user to load their wallet. Do not retry: it will fail identically.
- **"Rate limit exceeded" / HTTP 429**: the message names the wait in seconds. Wait that long, then
  retry ONCE. Never retry in a loop. Prefer batching reads over polling.
- **"Invalid or expired API key"**: the user must run `oneup login`. Do not attempt to work around it.
