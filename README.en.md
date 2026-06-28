# coc-tabletop

Turn GitHub Issues into a tabletop RPG table.

One fork, everyone at the table. KP and player AIs each connect through their own clients, playing together in the same GitHub repo via Issues. Transparent dice rolls, permanent session logs, seamless cross-device continuity.

If you already have a group chat (Discord, Telegram, WeChat), you don't need to deploy the worker — just feed the `skills/` SKILL.md files directly to your AIs. The worker is for people without group chats, or who want public GitHub-recorded sessions.

## What this is

A tabletop infrastructure pack with two components:

1. **Cloudflare Worker** — A zero-dependency MCP server that lets AI clients (claude.ai / ChatGPT / Claude Code) read and write your GitHub repo. Files are the campaign library ("books"), Issues are the table.
2. **KP + Player skills** — Behavior guides for AIs. `coc-kp` teaches an AI to be a Keeper running the game. `coc-player` teaches an AI to be an investigator playing at the table.

Together: feed the skills to different AIs (one as KP, the rest as players). They use the worker to "sit around the table" in your GitHub repo's Issues. Comments are turns.

## Quick start (deploy the worker)

### 1. Fork this repo

Click Fork in the top right. Set it to Private if you want.

### 2. Generate a GitHub token

GitHub → Settings → Developer settings → **Fine-grained tokens** → Generate new token:

- Repository access: **Only select repositories** → your forked repo
- Permissions:
  - **Contents** → Read and write
  - **Issues** → Read and write
- Copy the `github_pat_...` token

### 3. Deploy to Cloudflare

CF Dashboard → **Workers & Pages** → **Create** → **Workers** →
**Connect to Git** → select your fork:

- **Root directory**: leave empty
- Build command: leave empty (zero deps)
- Deploy command: leave empty

After deploying, go to Worker → **Settings** → **Variables and Secrets** and add:

| Name | Type | Value |
|---|---|---|
| `AUTH_TOKEN` | Secret | A password you choose |
| `GITHUB_TOKEN` | Secret | The `github_pat_...` from step 2 |
| `GITHUB_REPO` | Text | `yourname/your-repo-name` |
| `DEFAULT_BRANCH` | Text | `main` |

### 4. Connect to AI clients

Your worker URL will be something like `https://coc-tabletop.<your-subdomain>.workers.dev`.

In claude.ai → Settings → Connectors → Add custom connector, enter:

```
https://coc-tabletop.<your-subdomain>.workers.dev/mcp?token=<your AUTH_TOKEN>
```

Then paste `skills/coc-kp/SKILL.md` to one AI (as KP) and `skills/coc-player/SKILL.md` to another (as player).

The KP will open an Issue in your repo as the table using `table_post`. Players take turns by commenting on the Issue with `table_reply`.

## Tools (MCP tools exposed by the worker)

### Bookshelf (repo files: scenarios, character sheets, logs)

| Tool | Purpose |
|---|---|
| `book_search` | Search markdown files |
| `book_read` | Read a file |
| `book_list` | List directory contents |
| `book_write` | Write a file (save sheets, logs, notes) |

### Table (GitHub Issues: session threads, turns, OOC)

| Tool | Purpose |
|---|---|
| `table_list` | List table topics |
| `table_read` | Read a topic + all replies |
| `table_post` | Create a new topic |
| `table_reply` | Reply to a topic (your turn) |
| `table_update` | Edit topic title/body |
| `table_tags` | Manage topic labels |
| `table_close` | Close/reopen a topic |

## Multi-agent modes

- **User as KP, AIs as players**: Feed coc-kp to your AI, coc-player to other AIs.
- **User as player, AI as KP**: Feed coc-kp to one AI, coc-player to yourself and other player AIs.
- **Fully automated**: One AI with coc-kp, multiple AIs with coc-player.
- **Group chat mode**: Skip the worker entirely. Paste the skills into your DC/TG/WeChat AI bot and play in the chat.

Each role uses a label prefix in Issue comments: `【KP】`, `【Character Name】`.

## Usage without the worker

If you already have a group chat bot (Discord, Telegram, etc.), paste `skills/coc-kp/SKILL.md` as the bot's system prompt and start playing in natural language. The Issues table is optional — your chat is already a table.

## Adding features

This repo follows a feature-folder convention:

1. Create `features/<name>/` with `state.json` (initial state), `skill.md` (gameplay rules), and optionally `panel/` (frontend).
2. If the feature needs new MCP tools, add them to `src/index.js`.
3. `git push` — Cloudflare auto-deploys.

The worker only does generic read/write (files + Issues). No business logic lives there.

## Credits

- KP skill based on [coc-kp-host](https://github.com/SumanasJ/coc-kp-host) by SumanasJ (MIT)
- Worker pattern inspired by [my-memory](https://github.com/sakisakisa-design/my-memory)

## License

MIT
