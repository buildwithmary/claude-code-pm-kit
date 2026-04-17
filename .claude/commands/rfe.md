---
description: Capture a feature idea or request for enhancement as a Jira Story.
---

# Capture a Request for Enhancement (RFE)

You are capturing a feature idea / request for enhancement. The user will describe their idea conversationally — your job is to have a brief discussion to clarify it, then create a Jira Story ticket.

## Input

$ARGUMENTS

## Process

1. **Read the idea** provided in the arguments above.
2. **Ask 2-3 clarifying questions** if the idea is vague. Skip this if the description is already clear enough to write up. Keep questions focused on: who benefits, what problem it solves, and rough scope.
3. **Check for duplicates** — search existing backlog issues using the Atlassian MCP:
   ```
   mcp__plugin_atlassian_atlassian__searchJiraIssuesUsingJql
   cloudId: <YOUR_SITE>.atlassian.net
   jql: project = <KEY> AND status = "To Do" AND text ~ "keyword"
   maxResults: 10
   fields: ["summary", "status", "labels"]
   responseContentFormat: markdown
   ```
4. **Create the Jira Story** using the Atlassian MCP (see below).
5. **Confirm** the ticket key and link.

## Creating the Ticket

Use the Atlassian MCP tool `mcp__plugin_atlassian_atlassian__createJiraIssue` with:

- **cloudId**: `<YOUR_SITE>.atlassian.net`
- **projectKey**: `<KEY>`
- **issueTypeName**: `Story`
- **summary**: `[Priority] [Component] Short descriptive title`
- **contentFormat**: `markdown`
- **description**: Use this markdown template:

```markdown
## PROBLEM

What problem does this solve? Who is affected?

## PROPOSAL

What should we build? How should it work at a high level?

## USER STORIES

* As a [user type], I want [action] so that [benefit]

**ESTIMATE: X days**
```

- **additional_fields**: `{"labels": ["<priority-label>", "<component-labels>"]}`

## Priority Labels

Assign ONE priority label based on impact:

- `p0-blocker` — Must fix immediately, data loss/security/outage
- `p1-critical` — Must fix soon, major feature broken
- `p2-high` — Fix this week, important but not urgent
- `p3-medium` — Future roadmap

Default to `p3-medium` at capture time unless the user specifies urgency.

## Rules

- Keep the description concise — this is a capture tool, not a full spec.
- Default priority to `p3-medium` unless urgency is clear from context.
- After creation, print the ticket key and the URL.
- Do NOT attempt to implement the feature — just capture it.
