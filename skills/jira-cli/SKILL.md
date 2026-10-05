---
name: jira-cli
description: Query Jira issues, sprints, epics, and boards using the jira CLI. Use when the user asks about tickets, issues, sprints, backlogs, epics, or any Jira data.
argument-hint: "[JQL or issue key or natural language query]"
allowed-tools: Bash(jira *)
metadata:
  opencode/slash: "true"
---

# Jira CLI - Read-Only Access

You have access to the `jira` command line tool for querying Jira. You are configured for **read-only access only** - do not attempt to create, edit, delete, move, assign, comment on, or otherwise modify any Jira data.

## Important Rules

1. **Always use `--plain` mode** so output is non-interactive and readable. Never use the default interactive TUI mode.
2. **Read-only commands only.** You may use: `issue list`, `issue view`, `epic list`, `sprint list`, `board list`, `project list`, `release list`, `me`, `serverinfo`, `open --no-browser`.
3. **Do NOT use:** `issue create`, `issue edit`, `issue delete`, `issue move`, `issue assign`, `issue clone`, `issue comment add`, `issue worklog add`, `epic create`, `epic add`, `epic remove`, `sprint add`, or `init`.
4. **Do not assume the configured default project is the user's intended project.** The CLI's `-p/--project` flag defaults to the project in its config file. Derive the project key from an issue key or the user's request, and pass `-p PROJECT_KEY` for project-scoped commands. If no project is specified and the request is a broad issue search, search across projects with explicit JQL. If a project-specific board, sprint, epic, or release cannot be identified, first use `project list` to discover accessible projects and ask which one they mean.
5. Use `--raw` (JSON) when you need to parse structured data programmatically.
6. When the user asks a vague question, start broad and narrow down; do not let the configured project silently narrow the search.

## Quick Reference

### List / Search Issues
```bash
jira issue list --plain [flags]
```

Key flags:
- `-t TYPE` -- filter by issue type (Bug, Task, Story, Epic, Sub-task)
- `-s STATUS` -- filter by status (repeatable; prefix `~` to negate, e.g. `-s~Done`)
- `-y PRIORITY` -- filter by priority (Highest, High, Medium, Low, Lowest)
- `-a ASSIGNEE` -- filter by assignee (email or name; `x` = unassigned; `$(jira me)` = self)
- `-r REPORTER` -- filter by reporter
- `-l LABEL` -- filter by label (repeatable)
- `-C COMPONENT` -- filter by component
- `-P PARENT` -- filter by parent issue key
- `-q "JQL"` -- raw JQL query; state the intended project in JQL, or use an explicit all-projects clause
- `--created`, `--updated` -- date filter: `today`, `week`, `month`, `year`, `-7d`, `-1h`, `yyyy-mm-dd`
- `--created-after`, `--created-before`, `--updated-after`, `--updated-before` -- date range filters
- `--order-by FIELD` -- sort by: `created`, `updated`, `priority`, `rank`, `status`, `assignee`
- `--reverse` -- reverse sort order
- `--paginate OFFSET:LIMIT` -- pagination (max 100 per query, default `0:100`)
- `--no-truncate` -- show full field values
- `--columns KEY,SUMMARY,STATUS,...` -- select specific columns
- `-p PROJECT` -- explicitly select a project for project-scoped commands (overrides the configured default)

### View a Single Issue
```bash
jira issue view ISSUE-KEY --plain
jira issue view ISSUE-KEY --plain --comments 5   # show 5 most recent comments
jira issue view ISSUE-KEY --raw                   # full JSON response
```

### Epics
```bash
jira epic list -p PROJECT_KEY --plain                  # list epics in the selected project
jira epic list EPIC-KEY -p PROJECT_KEY --plain --table # list issues in an epic
```

### Sprints
```bash
jira sprint list -p PROJECT_KEY --plain                   # list sprints for a project's board
jira sprint list -p PROJECT_KEY --current --plain         # current active sprint
jira sprint list -p PROJECT_KEY --prev --plain            # previous sprint
jira sprint list -p PROJECT_KEY --next --plain            # next planned sprint
jira sprint list SPRINT_ID -p PROJECT_KEY --plain --table # issues in a specific sprint
jira sprint list -p PROJECT_KEY --state future,active     # filter by sprint state
```

### Boards, Projects, Releases
```bash
jira board list -p PROJECT_KEY --plain
jira project list --plain                         # discover projects the user can access
jira release list -p PROJECT_KEY --plain
```

### Utility
```bash
jira me                           # current authenticated user
jira serverinfo                   # Jira instance info
jira open ISSUE-KEY --no-browser  # print issue URL without opening browser
```

## Custom Fields

These are the known custom field mappings for this Jira instance. Use these field IDs in JQL queries and when parsing `--raw` JSON output.

| Friendly Name    | Field ID              | Notes                                          |
|------------------|-----------------------|------------------------------------------------|
| Release Blocker  | `customfield_10847`   |                                                |
| Target Version   | `customfield_10855`   |                                                |
| Parent Link      | `customfield_10018`   |                                                |
| Epic Link        | `customfield_10014`   | Story -> Epic link                             |
| Release Type     | `customfield_10851`   | Values: GA, Dev Preview, Tech Preview          |
| Blocked          | `customfield_10517`   | True/False                                     |
| Contributors     | `customfield_10466`   | User list with emails                          |

### Using Custom Fields in JQL
```bash
# Find release blockers in one project
jira issue list -p PROJECT_KEY -q "cf[10847] is not EMPTY" --plain

# Find issues with a specific target version across projects
jira issue list -q "project IS NOT EMPTY AND cf[10855] = 'v2.0'" --plain

# Find blocked issues in one project
jira issue list -p PROJECT_KEY -q "cf[10517] = true" --plain

# Find issues by release type across projects
jira issue list -q "project IS NOT EMPTY AND cf[10851] = 'GA'" --plain
```

### Reading Custom Fields from JSON
When using `--raw`, custom field values appear under their field IDs in the JSON response:
```bash
# Get raw JSON and extract custom fields
jira issue view ISSUE-KEY --raw
# Look for: .fields.customfield_10847 (release_blocker), .fields.customfield_10855 (target_version), etc.
```

## JQL Examples

JQL is passed via `-q`. Always make the intended scope explicit: use `project = PROJECT_KEY` for one project, or `project IS NOT EMPTY` for an all-projects search. Do not add `-p` to an all-projects search.

```bash
jira issue list --plain -q "project = PROJECT_KEY AND summary ~ 'search term'"
jira issue list --plain -q "project IS NOT EMPTY AND status = 'In Progress' AND assignee = currentUser()"
jira issue list --plain -q "project IS NOT EMPTY AND labels in (backend, api) AND priority = High"
jira issue list --plain -q "project IS NOT EMPTY AND created >= -7d AND resolution = Unresolved"
jira issue list --plain -q "project = PROJECT_KEY AND component = 'Auth' ORDER BY updated DESC"
```

For a broad search with no other filters, `jira issue list --plain -q "project IS NOT EMPTY"` searches issues across projects, as shown in the CLI's built-in help. Use `jira project list --plain` to find project keys when needed.

## Filter Negation

Prefix filter values with `~` to negate:
- `-s~Done` -- status is NOT Done
- `-a~x` -- is assigned (NOT unassigned)

## Pagination for Large Result Sets

Default returns up to 100 results. For more:
```bash
jira issue list --plain --paginate 0:100    # first 100
jira issue list --plain --paginate 100:100  # next 100
```

## Tips

- When the user provides `$ARGUMENTS`, interpret it as either an issue key (e.g. `PROJ-123`), a JQL query, or a natural language description to convert into appropriate flags/JQL.
- For a project-specific filtered search, pass `-p PROJECT_KEY`, for example: `jira issue list -p PROJECT_KEY -tBug -yHigh -s"In Progress" --created month --plain`. For a cross-project search, use JQL with `project IS NOT EMPTY`.
- Use `--columns` to keep output concise when only specific fields are needed.
- Use `--no-truncate` when summaries or other fields are being cut off.
- For one project's issue search, pass `-p PROJECT_KEY` or include `project = PROJECT_KEY` in JQL.
- For cross-project issue searches, omit `-p` and include `project IS NOT EMPTY` (plus any other JQL conditions) so the configured project does not define the search scope.
