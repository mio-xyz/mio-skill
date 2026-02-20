# Mio Tool Patterns

Detailed reference for each Mio MCP tool. The main [SKILL.md](../SKILL.md) covers *when* and *why* — this file covers *how*.

---

## list_categories

Discover which profile categories exist for this user.

**Parameters:** None

**Returns:** Array of `{ key, label }` objects. Keys are lowercase identifiers (e.g., `tech_stack`, `conventions`). Labels are human-readable names.

**Notes:**
- Call before `get_context` if you want to target specific `sources`
- Returns empty list if the user hasn't set up their profile yet
- Respects agent access controls — user may have hidden certain categories from your agent

---

## list_projects

Discover the user's active projects.

**Parameters:** None

**Returns:** Array of `{ id, name, description, type }` objects. IDs are UUIDs.

**Notes:**
- Only returns projects where the user is an active member
- Use the `id` field with `set_active_project` or as the `project` parameter in `get_context`
- `description` may be null

---

## get_context

Retrieve personalized user context. This is the primary tool — most sessions should call this at least once.

**Parameters:**

| Parameter   | Type     | Required | Description |
|-------------|----------|----------|-------------|
| `query`     | string   | Yes      | Natural language: what you're working on or need context for |
| `sources`   | string[] | No       | Category keys to include. Omit for all categories |
| `timeframe` | string   | No       | `"last_24_hours"`, `"last_7_days"`, or `"last_30_days"` |
| `project`   | string   | No       | Project UUID. Overrides the session active project |

**Returns:** Markdown-formatted text with sections per category, plus project context if a project is active.

**Patterns:**

```
# Targeted: just code-related context
get_context({ query: "Building a REST API", sources: ["tech_stack", "conventions"] })

# Broad: everything the user has
get_context({ query: "Planning a new feature" })

# Project-scoped: includes project-specific context
get_context({ query: "Refactoring the data layer", project: "550e8400-..." })

# Time-filtered: only recently updated categories
get_context({ query: "What changed recently?", timeframe: "last_7_days" })
```

**Notes:**
- If `set_active_project` was called earlier, project context is included automatically — no need to pass `project`
- Passing `project` explicitly overrides the session active project for that single call
- Response includes both shared project entries and personal notes

---

## set_active_project

Set the active project for this session. Once set, all `get_context` calls automatically include that project's context.

**Parameters:**

| Parameter | Type          | Required | Description |
|-----------|---------------|----------|-------------|
| `project` | string / null | Yes      | Project UUID from `list_projects`, or `null` to clear |

**Notes:**
- Sessions expire after 1 hour of inactivity
- Call once early in the conversation — avoids passing `project` to every `get_context` call
- Pass `null` to clear the active project if switching to non-project work

---

## log_conversation

Record a conversation summary. Summaries are stored for the user to review in their Mio dashboard.

**Parameters:**

| Parameter   | Type     | Required | Description |
|-------------|----------|----------|-------------|
| `summary`   | string   | Yes      | 1–5000 chars. What was discussed, decided, or learned |
| `takeaways` | string[] | No       | Max 20 items, 500 chars each. Key facts or preferences learned |
| `tags`      | string[] | No       | Max 20 items, 100 chars each. Topic tags for categorization |

**Patterns:**

```
# After a debugging session
log_conversation({
  summary: "Debugged CORS issue in API gateway. Root cause was missing Access-Control-Allow-Credentials header when using cookie-based auth.",
  takeaways: ["API uses cookie-based auth, not Bearer tokens", "CORS config lives in middleware, not per-route"],
  tags: ["debugging", "cors", "api"]
})

# After an architecture discussion
log_conversation({
  summary: "Decided to use event sourcing for the audit log instead of CDC. Simpler to implement and sufficient for current scale.",
  takeaways: ["Team chose event sourcing over CDC for audit", "Scale threshold for reconsidering: 10k events/sec"],
  tags: ["architecture", "audit-log"]
})
```

**Notes:**
- Be specific in summaries — "Migrated auth from JWT to sessions" is useful; "discussed authentication" is not
- Takeaways should be facts that would help a future agent understand the user better
- Call once at the end of a substantive conversation, not after every exchange

---

## update_profile

Suggest a profile update. Suggestions go to a review queue — the user approves or rejects each one.

**Parameters:**

| Parameter    | Type   | Required | Description |
|--------------|--------|----------|-------------|
| `category`   | string | Yes      | 1–100 chars. Category key from `list_categories` |
| `suggestion` | string | Yes      | 1–5000 chars. The suggested content or update |
| `reason`     | string | No       | Max 1000 chars. Why you think this is correct |

**Patterns:**

```
# User explicitly mentions a tool change
update_profile({
  category: "tech_stack",
  suggestion: "Uses Bun as package manager and JavaScript runtime (migrated from npm)",
  reason: "User said 'we switched to Bun last week' during conversation"
})

# User describes their team's workflow
update_profile({
  category: "conventions",
  suggestion: "All PRs require at least 2 approvals. Squash merge only. Branch naming: type/ticket-description",
  reason: "User described their PR workflow in detail while setting up CI"
})
```

**Notes:**
- Always include `reason` — it helps the user understand why you're suggesting the change
- Only one suggestion per category per conversation to avoid spamming the review queue
- Never assume a suggestion was accepted — the user must explicitly approve it
