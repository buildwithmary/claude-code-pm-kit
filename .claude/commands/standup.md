---
description: Use when running daily standups, prioritizing work, or checking project status. Synthesizes Jira backlog with git activity to recommend what to work on next. Say "/standup", "what should I work on?", or "show project status".
---

# Product Owner Skill

A Claude-native product owner that provides daily status, priority recommendations, and work-in-progress awareness. Adapted for a solo developer working in continuous flow (no sprints).

## Philosophy

- **Recommend, don't dictate**: Present priorities; user decides what to act on
- **Continuous flow**: No sprints or ceremonies — priorities adjust daily
- **Data-driven**: Use Jira state + git activity to surface what matters
- **Concise**: Keep reports scannable — no walls of text

## When to Use

| Trigger | Use Case |
|---------|----------|
| `/standup` | Daily status report with recommendations |
| "what should I work on?" | Priority-ranked recommendations |
| "project status" | Current state overview |
| "what's blocking?" | Stale work and blocker analysis |

## Data Sources

### Jira (via Atlassian MCP)

All Jira queries use `cloudId: "<YOUR_SITE>.atlassian.net"` and `responseContentFormat: "markdown"`.

| Tool | Purpose |
|------|---------|
| `searchJiraIssuesUsingJql` | Backlog issues, filtered by priority/status/label |
| `getJiraIssue` | Individual issue details |
| `transitionJiraIssue` | Move issues between statuses |

**Key JQL queries:**
```
# All open items by priority
project = <KEY> AND status = "To Do" ORDER BY created DESC

# In-progress items
project = <KEY> AND status = "In Progress"

# Recently completed
project = <KEY> AND status = "Done" AND updated >= -7d ORDER BY updated DESC

# P0/P1 critical items
project = <KEY> AND status != "Done" AND (labels = "p0-blocker" OR labels = "p1-critical")
```

### Git (via CLI)

| Command | Purpose |
|---------|---------|
| `git log --oneline -20` | Recent commits on current branch |
| `git log --oneline --since="yesterday" main` | What shipped since last session |
| `git branch -a` | Detect stale branches |
| `git status` | Uncommitted work |
| `gh pr list` | Open PRs and their states |

### Context DB (`.workspace-temp/context-db.json`)

Read at the start of every standup for:
- Recent session history (what was worked on, what shipped)
- Decisions made in previous sessions
- Estimate accuracy trends
- Patterns observed over time

---

## Priority Classification

### How Labels Map to Priority Tiers

| Tier | Jira Labels | Action |
|------|-------------|--------|
| **CRITICAL** | `p0-blocker`, `data-corruption-risk`, `silent-failure` | Stop current work, fix immediately |
| **HIGH** | `p1-critical` | Prioritize this session |
| **MEDIUM** | `p2-high` | This week's focus after HIGH items |
| **LOW** | `p3-medium` | Future roadmap, pick up when ahead |

### WSJF-Lite Scoring

For items at the same priority tier, break ties with a simplified WSJF:

```
Score = (User Impact + Time Sensitivity + Risk Reduction) / Effort

Where:
- User Impact: How many users or how visibly affected (1-5)
- Time Sensitivity: Is there a deadline or dependency? (1-5)
- Risk Reduction: Does this prevent data loss, security issues, or silent failures? (1-5)
- Effort: Rough size — 1 (hours), 2 (1-2 days), 3 (3-5 days), 5 (week+)
```

### Auto-Scoring from Labels

| Label | Component | Value |
|-------|-----------|-------|
| `p0-blocker` | Time Sensitivity | 5 |
| `p1-critical` | Time Sensitivity | 4 |
| `data-corruption-risk` | Risk Reduction | 5 |
| `silent-failure` | Risk Reduction | 4 |
| `security` | Risk Reduction | 5 |
| `quick-win` | Effort | 1 |
| `polish` | User Impact | 2 |
| `technical-debt` | Risk Reduction | 3 |

---

## Standup Report

### Process

1. **Read context DB**: `.workspace-temp/context-db.json` — recent sessions, decisions, patterns, estimate accuracy
2. **Check git state**: current branch, uncommitted work, recent commits
3. **Check open PRs**: any waiting for merge or review
4. **Query Jira**: in-progress items, then full backlog by priority
5. **Query recently completed**: what shipped in last 7 days
6. **Detect stale work**: branches without PRs, old PRs, items stuck in progress
7. **Apply patterns**: use estimate history to adjust effort scores (e.g., if quick-wins consistently take half the estimate, factor that in)
8. **Generate recommendations**: top 3 actionable next steps

### Report Format

```
╔═══════════════════════════════════════════════════════════╗
║  Daily Status: <Project>                                  ║
║  <date>                                                   ║
╠═══════════════════════════════════════════════════════════╣

Since Last Session:
  - <what was shipped/merged recently>
  - <commits, PRs merged>

Current Work:
  - Branch: <current branch>
  - Uncommitted: <yes/no>
  - Open PRs: <list with status>

IN PROGRESS (<count>):
  * KEY-XXX: <summary> — <status note>

CRITICAL / P0-P1 (<count>):
  * KEY-XXX: <summary> [p0-blocker]

READY TO PICK UP (<count>, ranked by WSJF-lite):
  1. KEY-XXX: <summary> [p2-high, quick-win] — Score: X.X
  2. KEY-XXX: <summary> [p2-high] — Score: X.X
  3. KEY-XXX: <summary> [p3-medium] — Score: X.X

Stale Work:
  * <branch with no PR, old PR, stuck item>

Outstanding Non-Code:
  * <changelog not written, blog post pending, etc.>

Recommendations:
  1. <actionable next step>
  2. <actionable next step>
  3. <actionable next step>

╚═══════════════════════════════════════════════════════════╝
```

---

## Stale Work Detection

### Stale PRs

**Criteria:**
- Open >48h with no review activity
- Approved but not merged >24h
- Changes requested but no updates >48h

**Action:** Flag and suggest merging or closing.

### Stale Branches

**Criteria:**
- Branch with commits not on main
- No open PR associated
- Last commit >48h ago

**Action:** Prompt to create PR or delete branch.

### Stuck In-Progress Items

**Criteria:**
- Jira status "In Progress" but no git branch matching the ticket key
- In Progress >7 days with no recent commits

**Action:** Flag and suggest moving back to To Do or completing.

---

## "What Should I Work On?" Response

When the user asks what to work on next:

1. Check if there's unfinished work (open PR, uncommitted changes, in-progress Jira item)
2. If yes: recommend finishing that first
3. If no: rank backlog items by tier, then WSJF-lite within tier
4. Present top 3 recommendations with reasoning
5. Note any non-code tasks (changelog, blog posts, docs)

**Format:**
```
Based on current state, I recommend:

1. [FINISH] KEY-XXX — you have an open PR for this, merge it
2. [HIGH] KEY-XXX — p1-critical, quick-win, clears a blocker
3. [MEDIUM] KEY-XXX — p2-high, good WSJF score (impact: 4, effort: 1)

Also pending:
  - Changelog not yet written
```

---

## Error Handling

| Scenario | Handling |
|----------|----------|
| Atlassian MCP not connected | Prompt user to run `/mcp` to authenticate |
| No Jira issues found | Note clean backlog, suggest creating new work |
| No git remote | Skip PR checks, show local state only |
| Empty backlog | "Clean slate — what's the first task?" |
| No context DB | Skip history section, note first standup |

## Rules

- Never fabricate Jira data — only report what the MCP returns
- Keep the report under 40 lines for scannability
- Always check git state first — unfinished local work takes priority over new backlog items
- Don't suggest starting new work if there's an open PR waiting to merge
- Include non-code tasks (changelog, blog posts) when known from session context
