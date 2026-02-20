# Mio Agent Skill

Personal context for AI agents. Mio remembers your preferences, tech stack, coding conventions, and project details so your AI agents don't have to re-ask. This skill teaches agents when and how to use Mio's MCP tools automatically.

## Prerequisites

- A [Mio](https://mio.xyz) account with a profile set up
- Mio MCP server connected to your agent ([Hub connections page](https://hub.mio.xyz/connections))

## Install

```bash
npx skills add mio-xyz/mio-skill
```

## What the skill does

- **Loads your context at session start** — agents call `get_context` to get your preferences before writing code or making decisions
- **Project awareness** — agents discover your projects and scope context to the one you're working on
- **Logs conversations** — at the end of substantive sessions, agents record what was discussed and key takeaways
- **Suggests profile updates** — when you mention new preferences, agents suggest updates for your review in the Hub
- **Graceful degradation** — works even if only some Mio MCP tools are available

## Available MCP tools

| Tool | Purpose |
|------|---------|
| `get_context` | Retrieve user preferences, tech stack, conventions |
| `list_categories` | Discover available profile sections |
| `list_projects` | Discover user's projects |
| `set_active_project` | Set project scope for the session |
| `log_conversation` | Record conversation summaries |
| `update_profile` | Suggest profile updates for user review |

## Links

- [Mio Hub](https://hub.mio.xyz) — manage your profile and connections
- [mio.xyz](https://mio.xyz) — learn more about Mio
