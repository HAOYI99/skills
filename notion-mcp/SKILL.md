---
name: notion-mcp
description: "Use for Notion MCP setup and operations: initialize Notion config, search Notion, create/update pages or tasks, sync tasks to Notion, add comments. Trigger on \"Notion\", \"setup Notion\", \"configure Notion\", \"sync to Notion\", \"create a Notion task\", \"update Notion\", or \"comment in Notion\"."
---

# Notion

Use this skill when the user explicitly wants a Notion read or write, or needs to initialize Notion config for the first time.

## Initialization (required once per workspace)

Run when `.env.local` has no Notion values, or user asks to set up / reconfigure Notion.

### Step 1 — Check Notion MCP

Use `tool_search` to probe for Notion MCP tools (search for `notion`).

- **Tools found** → MCP ready, proceed to interview.
- **No tools found** → tell user:

  > Notion MCP is not connected. Follow the official setup guide:
  > **https://developers.notion.com/guides/mcp/get-started-with-mcp**
  >
  > Once done, reload your MCP tools and run setup again.

  Wait for user confirmation that MCP is connected before continuing.

### Step 2 — Interview

Interview the user — **one question at a time** — until all values are captured. Then write to `.env.local`.

### Interview sequence

1. Do you work with a single default Notion project, or pick a project per task?
   - `single` → collect project page URL → `NOTION_PROJECT_PAGE_URL`, set `NOTION_PROJECT_MODE=single`.
   - `prompt` → set `NOTION_PROJECT_MODE=prompt`; ask project page at sync time.

2. What is the Notion database / data source ID for your tasks? (format: `collection://...` or plain database ID)
   - Save as `NOTION_TASKS_DATA_SOURCE_ID`.

3. Do you have any custom preferences for every row the agent creates? For example:
   - Always set a `Source` property to `AI`
   - Always fill a `Start Date` property with today's date
   - Always apply a `Tags` value of `AI`
   - Or any other field/value pair
   - Collect all stated preferences as `NOTION_CUSTOM_PROP_<PROPERTY_NAME>=<value>` pairs.
   - Use `NOTION_CUSTOM_PROP_<PROPERTY_NAME>=__today__` for date fields the agent should fill automatically.

4. Any other preferences? Keep asking until user says no.

5. Do you want to review what will be written before the agent pushes to Notion?
   - `yes` → set `NOTION_REVIEW_BEFORE_WRITE=true`; agent shows a preview and waits for confirmation on every create/update.
   - `no` → set `NOTION_REVIEW_BEFORE_WRITE=false`; agent writes immediately.

After all answers: write to `.env.local` (git-ignored), confirm what was saved.

`.env.local` format:

```env
# single = one default project | prompt = ask per task
NOTION_PROJECT_MODE=single
NOTION_PROJECT_PAGE_URL=https://www.notion.so/<your-project-page-id>
NOTION_TASKS_DATA_SOURCE_ID=collection://<your-data-source-id>

# true = show preview and wait for confirmation before every write
NOTION_REVIEW_BEFORE_WRITE=true

# Custom properties applied to every AI-created row
# NOTION_CUSTOM_PROP_Source=AI
# NOTION_CUSTOM_PROP_Tags=AI
# NOTION_CUSTOM_PROP_Start_Date=__today__
```

## Core rules

- Do not write to Notion unless the user asks.
- Search for an existing matching page before creating a new one.
- Use exact property names from the target data source schema.
- Apply every `NOTION_CUSTOM_PROP_*` entry on each AI-created row. Skip silently only if the property does not exist in the schema — ask one question first to confirm.
- Pass relation and multi-select values as JSON text strings, not raw arrays.

## When to use Notion MCP

Use Notion MCP when the user asks to:

- create a Notion task, page, or database item
- update a Notion page or task status
- search Notion for an existing task or document
- sync a task record into Notion
- add or manage comments in Notion
- prepare a Notion-ready summary and then write it

Do not use Notion MCP for local-only updates or draft summaries with no Notion write.

## Workflow

1. **Confirm outcome.** Distinguish `search` / `draft only` / `create` / `update` / `comment` / `sync`.

2. **Load config.** Read `.env.local`. Missing required values → run init interview before continuing.
   - `NOTION_PROJECT_MODE=prompt` → ask which project page for this task.

3. **Check for duplicates.** Search Notion first. Reuse or update existing page when clearly the same item.

4. **Write to Notion.**
   - If `NOTION_REVIEW_BEFORE_WRITE=true`: show a preview of all fields and values, wait for explicit user confirmation before writing.
   - Apply all `NOTION_CUSTOM_PROP_*` values. Resolve `__today__` to current date.

5. **Report.** State what changed in Notion.

## Completion checks

- Requested Notion action completed or specific blocker identified.
- No duplicate page created when a matching page already existed.
- All `NOTION_CUSTOM_PROP_*` entries applied to the created/updated row.
- Report states exactly what changed.