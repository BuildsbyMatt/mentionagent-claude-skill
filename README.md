# MentionAgent Claude skill: link building outreach from Claude Code, Hermes Agent, Cursor, OpenCode and OpenClaw

A Claude Code plugin (and a standalone Agent Skill) that runs [MentionAgent](https://mentionagent.ai) link building outreach from your agent: review the drafts it wrote, send the batch, answer the publishers who replied, and close placements, without opening the dashboard.

It wraps the MentionAgent MCP server (`https://mentionagent.ai/mcp`, 38 tools) with a skill that knows the daily loop, which two tools send real email, which two spend credits, and where to stop and ask you.

Works in Claude Code, Cursor, Windsurf, or any MCP client that can set a request header.

## What it does

- **Triage**: which sites have drafts waiting, which threads need an answer.
- **Draft review in bulk**: reads every pending draft and flags only the ones that look wrong, then edits or discards them.
- **Send once**: approves the batch you have seen.
- **Inbox in one pass**: answers publisher replies, and hands placement proposals to MentionAgent's own reply writer, which crawls the publisher's site and picks the page and paragraph to ask for.
- **Closes the loop**: records a placement as done once the link is live.
- **Campaign changes in plain English**, shown as a plan before anything moves.

## Install as a Claude Code plugin

```
claude plugin marketplace add BuildsbyMatt/mentionagent-claude-skill
claude plugin install mentionagent@mentionagent-claude-skill
```

Then sign in: run `/mcp` in Claude Code, pick `mentionagent` and choose Authenticate. A browser window opens on the MentionAgent consent page, you click Allow, and there is no key to paste or store. The connection is listed and revoked in the dashboard under Settings, Agent access.

Prefer an API key (a headless box, CI)? Add the server by hand with a bearer header instead, as under "Install the skill on its own" below. Keys are created under Settings, Agent access and are shown once.

## Install in Cursor

Cursor reads Agent Skills from `.cursor/skills/` (project) or `~/.cursor/skills/` (global), and also from `.claude/skills/` for compatibility. Copy the skill folder in, then add the server to `.cursor/mcp.json`:

```
mkdir -p .cursor/skills
cp -r skills/link-building-outreach .cursor/skills/
```

```json
{
  "mcpServers": {
    "mentionagent": {
      "url": "https://mentionagent.ai/mcp",
      "headers": { "Authorization": "Bearer ma_live_..." }
    }
  }
}
```

Type `/link-building-outreach` in the Agent chat to invoke it. Full walkthrough: https://mentionagent.ai/cursor-seo-skill/

## Install in OpenCode

OpenCode reads Agent Skills from `.opencode/skills/` (project) or `~/.config/opencode/skills/` (global), and also from `.claude/skills/` and `.agents/skills/`, so a copy installed for Claude Code or Cursor is already visible. Copy the folder in, then add the server as a remote MCP entry in `opencode.json` and sign in with OAuth (a browser window opens on the MentionAgent consent page):

```
mkdir -p .opencode/skills
cp -r skills/link-building-outreach .opencode/skills/
```

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "mentionagent": { "type": "remote", "url": "https://mentionagent.ai/mcp", "enabled": true }
  },
  "permission": {
    "mentionagent_approve_batch": "ask",
    "mentionagent_send_reply": "ask",
    "mentionagent_redirect_thread": "ask"
  }
}
```

```
opencode mcp auth mentionagent
opencode mcp list
```

The `permission` rules matter: OpenCode defaults to `allow` and MCP tools are registered as `mentionagent_<tool>`, so without them the two tools that send email, and `redirect_thread`, which opens a thread to an address taken from an email, run without a prompt. Never start OpenCode with `--auto` in a project that has this server enabled. To use an API key instead of OAuth, set `"oauth": false` on the server and add `"headers": { "Authorization": "Bearer ma_live_..." }` with your key. There is no slash command: ask the agent what is waiting on MentionAgent and it loads the skill through its `skill` tool. Full walkthrough: https://mentionagent.ai/opencode-seo-skill/

## Install in OpenClaw

OpenClaw reads Agent Skills from `<workspace>/skills/` and `~/.agents/skills/` (the `SKILL.md` can sit up to six folders deep). Install the folder, register the server, sign in with OAuth (a browser window opens on the MentionAgent consent page), then probe:

```
openclaw skills install ./skills/link-building-outreach
openclaw mcp add mentionagent \
  --url https://mentionagent.ai/mcp \
  --transport streamable-http \
  --auth oauth --no-probe
openclaw mcp login mentionagent
openclaw mcp probe mentionagent
```

To use an API key instead of OAuth, drop `--auth oauth` and pass `--header "Authorization=Bearer ma_live_..."` (the flag takes `KEY=VALUE`; a colon is rejected). `openclaw mcp doctor` will then flag the literal token, which is the reason to prefer OAuth on a shared Gateway. The skill answers to `/link-building-outreach` in any connected channel. Full walkthrough: https://mentionagent.ai/openclaw-seo-skill/

## Install in Hermes Agent

You need Hermes Agent 0.21.6 or newer, and Node.js 20.18.1 or newer with `npx` on your `PATH`.

From the Hermes plugin catalog:

```bash
hermes plugins install mentionagent
hermes plugins enable mentionagent
```

Or straight from GitHub:

```bash
hermes plugins install BuildsbyMatt/mentionagent-claude-skill --no-enable
hermes plugins enable mentionagent
```

Start a new session. The first time the server starts, your browser opens the MentionAgent consent page. Sign in with the account you use on [mentionagent.ai](https://mentionagent.ai) and click Allow. The token is kept and refreshed, so you sign in once. If the tools don't show up after you sign in, run `/reload-mcp`. The connection is listed and revoked in the dashboard under Settings, Agent access.

The Hermes package is `plugin.json`, `mcp.json` and the skill in `skills/link-building-outreach/`. It ships no code of its own. Before you install, know that:

- `mcp.json` runs the pinned `mcp-remote@0.1.49` through `npx`, which downloads it from npm on first use. It opens a local callback port for the sign-in and stores the OAuth tokens in `~/.mcp-auth`.
- Every tool call goes to `mentionagent.ai/mcp` and acts on your MentionAgent account. The server decides the tool set at runtime, so the pinned commit doesn't cover it.
- `plugin.json` marks the server `untrusted`. Hermes asks before every tool the server doesn't mark read-only, and refuses those tools in cron and other unattended runs.
- Two tools send real email: `approve_batch` sends an outreach batch from your warmed-up inbox and `send_reply` answers a publisher. Two tools spend MentionAgent credits: `trigger_run` and `draft_reply`.
- The package sends no telemetry.

On a server or another machine without a browser, skip the plugin and add the server to `~/.hermes/config.yaml` with an API key from Settings, Agent access in the [MentionAgent dashboard](https://mentionagent.ai/dashboard/):

```yaml
mcp_servers:
  mentionagent:
    url: "https://mentionagent.ai/mcp"
    headers:
      Authorization: "Bearer ${MENTIONAGENT_API_KEY}"
```

Put `MENTIONAGENT_API_KEY=ma_live_...` in `~/.hermes/.env`. The key works like a password for your MentionAgent account, so don't paste it into a chat.

## Install the skill on its own

Copy `skills/link-building-outreach/` into `~/.claude/skills/` (personal) or `.claude/skills/` in a project, then connect the MCP server yourself:

```
claude mcp add --transport http mentionagent https://mentionagent.ai/mcp \
  --header "Authorization: Bearer ma_live_..."
```

Or, in any client that takes a JSON config:

```json
{
  "mcpServers": {
    "mentionagent": {
      "type": "http",
      "url": "https://mentionagent.ai/mcp",
      "headers": { "Authorization": "Bearer ma_live_..." }
    }
  }
}
```

## First run

Ask your agent: *"Connect to MentionAgent and show me what is waiting."* It calls `get_status`, lists your sites, and tells you what it can and cannot do on your behalf.

## What the key can reach

The 38 published tools and nothing else. It cannot see mailbox credentials, billing, domain transfer or account deletion; those stay behind a login. Two tools send email (`approve_batch`, `send_reply`) and both are written to be shown to you first. `send_reply` has no recipient field: the address comes from the thread. The one tool that takes an address is `redirect_thread`. It sends nothing itself, but it opens a thread to that address, and `send_reply` on the new thread writes to it. That address usually comes from an inbound email, so the skill shows it to you, with the sentence it came from, and waits for you to confirm before redirecting. `set_run_settings` with `autoSend: true` lets future batches go out with no review; the skill only sets it when you ask for it.

Full tool reference and limits: https://mentionagent.ai/mcp/

## Files

```
.claude-plugin/plugin.json          Claude Code plugin manifest
.mcp.json                           the MentionAgent MCP server for Claude Code, OAuth sign-in
plugin.json                         Agent Plugins v1 manifest (Hermes Agent)
mcp.json                            the MentionAgent MCP server for Hermes, OAuth through mcp-remote
skills/link-building-outreach/      the skill (SKILL.md)
```

The canonical copy of this repo lives in the MentionAgent monorepo under `claude-skill/` and is published here on each change.

MIT licensed.
