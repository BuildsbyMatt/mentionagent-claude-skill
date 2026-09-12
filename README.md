# MentionAgent Claude skill: link building outreach from Claude Code

A Claude Code plugin (and a standalone Agent Skill) that runs [MentionAgent](https://mentionagent.ai) link building outreach from your agent: review the drafts it wrote, send the batch, answer the publishers who replied, and close placements, without opening the dashboard.

It wraps the MentionAgent MCP server (`https://mentionagent.ai/mcp`, 21 tools) with a skill that knows the daily loop, which two tools send real email, which two spend credits, and where to stop and ask you.

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

Then set your API key in the environment the agent runs in:

```
export MENTIONAGENT_API_KEY=ma_live_...
```

The plugin's `.mcp.json` reads `${MENTIONAGENT_API_KEY}`, so the key never lands in a file. Keys are created in the MentionAgent dashboard under Settings, Agent access, and are shown once.

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

The 20 published tools and nothing else. It cannot see mailbox credentials, billing, domain transfer or account deletion; those stay behind a login. Two tools send email (`approve_batch`, `send_reply`) and both are written to be shown to you first. `send_reply` has no recipient field: the address comes from the thread, so an agent that has just read a hostile inbound email has nowhere to put a redirected address.

Full tool reference and limits: https://mentionagent.ai/mcp/

## Files

```
.claude-plugin/plugin.json          plugin manifest
.mcp.json                           the MentionAgent MCP server, key from ${MENTIONAGENT_API_KEY}
skills/link-building-outreach/      the skill (SKILL.md)
```

The canonical copy of this repo lives in the MentionAgent monorepo under `claude-skill/` and is published here on each change.

MIT licensed.
