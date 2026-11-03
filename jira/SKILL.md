---
name: jira
description: "Manage Jira work through Atlassian MCP: fetch and search issues, create or update tasks, assign, comment, transition, and manually reconcile repository tasks. Use when the user asks to create, sync, fetch, search, update, or otherwise manage Jira issues for the current repository."
---

# Jira

## Workflow

1. **Check MCP availability.** Use tool discovery to verify the Atlassian Jira tool needed for this request and load its schema before calling it. If unavailable, stop Jira work and tell the user to connect the [official Atlassian remote MCP server](https://support.atlassian.com/atlassian-ai-gateway/docs/get-started-with-the-atlassian-remote-mcp-server/) at `https://mcp.atlassian.com/v2/mcp`, complete its OAuth sign-in, and reload tools. Resume only after the tools are available. Do not edit MCP configuration or request, store, or write credentials.

2. **Resolve the repository project.** Find the Git root with `git rev-parse --show-toplevel` and read `<root>\.jira\config.json` if present. Its project mapping is:

   ```json
   { "projectKey": "PROJ" }
   ```

   If the mapping is missing or invalid, use `jira_search_projects` with the repository name only to find candidates, then ask the user to choose the project key. After the user confirms, create or update `.jira/config.json`, preserving any other settings. Never infer or silently select a project. Use a configured mapping for reads; before creating issues, use a project key stated in the current request or ask the user to confirm the configured key.

3. **Perform only the requested action.** Resolve required inputs from the user request, tool schema, and Jira metadata. Ask for any missing or ambiguous value before writing; never guess.
   - **Fetch/search:** Fetch requires an issue key; search requires a clear query and scope. Use `jira_get_issue` for a known key and `jira_search` for JQL. Request only needed fields and scope to the confirmed project when appropriate.
   - **Create:** Requires an explicit project, issue type, summary, and every required field returned by the create schema. Check for likely duplicates with `jira_search`; load issue types and `jira_get_create_fields`, then retrieve allowed options when needed. Ask for missing values.
   - **Batch create:** Each issue needs its explicit project, type, summary, and required fields. Validate with `jira_batch_create_issues` using `validate_only=true`, show the proposed issues, and get user approval before writing.
   - **Update:** Requires an issue key and the requested field changes. Fetch the current issue first, change only those fields, and return only fields needed to confirm the result.
   - **Transition:** Requires an issue key, target status, a valid transition ID from `jira_get_transitions`, and every field required by that transition. For close/resolve transitions, resolution is mandatory: use a configured value supplied by the user or verify that the workflow sets it automatically. If it is not auto-set, ask the user for an allowed value and pass it through `fields`. After transitioning, fetch `status,resolution` with `jira_get_issue` and verify resolution is set. Never guess a value or proceed when resolution cannot be ensured.
   - **Assign:** Requires an issue key and a confirmed assignee. Use `jira_search_assignable_users` to disambiguate a person, then `jira_assign_issue`.
   - **Comment:** Requires an issue key and the requested comment text. For JSM, require explicit visibility; do not make a customer-visible comment unless the user requests it.
   - Do not delete or move issues unless the user explicitly requests the action and confirms the exact issue and target.

4. **Reconcile tasks only on request.** For manual repo↔Jira sync, clarify the direction, task source, and scope if unclear. Read only the user-selected files or task list; do not treat every TODO in the repository as authoritative. Search Jira for matches before proposing creates. Show a concise change list and get approval before multi-issue writes. Write back to repository files only when requested. Never run background or recurring sync.

5. **Report completion.** State which issues were fetched, created, or changed, include their keys and links when returned, and identify every requested item that remains blocked or unmatched. Do not claim sync is complete until all in-scope items are reconciled.

Examples: “Fetch `PROJ-123`”, “Create a Jira task for …”, “Sync the selected checklist with Jira”.
