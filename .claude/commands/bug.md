---
description: Capture a bug report as a Jira Bug ticket.
---

# Capture a Bug Report

You are capturing a bug report. The user will describe the issue conversationally — your job is to extract the key details, then create a Jira Bug ticket.

## Input

$ARGUMENTS

## Process

1. **Read the bug description** provided in the arguments above.
2. **Ask 1-2 clarifying questions** only if critical information is missing (e.g., which service, how to reproduce). Skip if the description is clear enough.
3. **Check for duplicates** — search existing backlog issues using the Atlassian MCP:
   ```
   mcp__plugin_atlassian_atlassian__searchJiraIssuesUsingJql
   cloudId: <YOUR_SITE>.atlassian.net
   jql: project = <KEY> AND status = "To Do" AND text ~ "keyword"
   maxResults: 10
   fields: ["summary", "status", "labels"]
   responseContentFormat: markdown
   ```
4. **Create the Jira Bug ticket** using the Atlassian MCP (see below).
5. **Confirm** the ticket key and link.

## Creating the Ticket

Use the Atlassian MCP tool `mcp__plugin_atlassian_atlassian__createJiraIssue` with:

- **cloudId**: `<YOUR_SITE>.atlassian.net`
- **projectKey**: `<KEY>`
- **issueTypeName**: `Bug`
- **summary**: `[Severity] [Component] Short descriptive title`
- **contentFormat**: `markdown`
- **description**: Use this markdown template:

```markdown
## SYMPTOMS

What the user sees — error messages, unexpected behavior

## STEPS TO REPRODUCE

1. Step 1
2. Step 2
3. Expected vs actual result

## LIKELY ROOT CAUSE

Best guess at what is wrong
```

- **additional_fields**: `{"labels": ["<severity-label>", "<component-labels>"]}`

## Severity Labels

Assign ONE severity label based on impact:

- `p0-blocker` — Data loss, security vulnerability, or complete feature outage
- `p1-critical` — Major feature broken, no workaround, affects most users
- `p2-high` — Feature partially broken, workaround exists
- `p3-medium` — Cosmetic, minor UX issue, or edge case

## Rules

- Keep the bug report concise — this is a capture tool, not a full investigation.
- After creation, print the ticket key and the URL.
- Do NOT attempt to fix the bug — just capture it.
