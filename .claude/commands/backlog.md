---
description: Show Jira Backlog
---

# Show Jira Backlog

Fetch and display the current project backlog from Jira using the Atlassian MCP.

## Process

1. Query backlog issues using:
   ```
   mcp__plugin_atlassian_atlassian__searchJiraIssuesUsingJql
   cloudId: <YOUR_SITE>.atlassian.net
   jql: project = <KEY> AND status = "To Do" ORDER BY created DESC
   maxResults: 50
   fields: ["summary", "status", "labels"]
   responseContentFormat: markdown
   ```

2. Display the results grouped by priority:
   - **P0 Blocker** — labels containing `p0-blocker`
   - **P1 Critical** — labels containing `p1-critical`
   - **P2 High** — labels containing `p2-high`
   - **P3 Medium** — labels containing `p3-medium`
   - **Other** — anything that doesn't match the above

3. Format as a table with columns: Key, Summary, Labels (excluding priority label).

4. Show the total count at the top.
