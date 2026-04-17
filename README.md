# claude-code-pm-kit

A lightweight product management toolkit for solo developers using [Claude Code](https://docs.anthropic.com/en/docs/claude-code). Gives you daily standups, backlog management, ticket creation, decision logging, and cross-session context — all through slash commands.

No sprints. No ceremonies. No story points. Just continuous flow with intelligent recommendations.

## What's in the kit

| Skill | Command | What it does |
|-------|---------|-------------|
| **Standup** | `/standup` | Daily status report — synthesizes Jira backlog + git state, recommends what to work on next |
| **Reflect** | `/reflect` | Log decisions, patterns, or estimate accuracy to a context DB for cross-session learning |
| **RFE** | `/rfe` | Capture a feature idea as a Jira Story with structured description |
| **Bug** | `/bug` | Capture a bug report as a Jira Bug with reproduction steps |
| **Backlog** | `/backlog` | Show your Jira backlog grouped by priority |

Plus:
- **Context DB** — a JSON file that persists session history, decisions, and patterns across conversations
- **ADR template** — architecture decision records that live in your repo forever
- **End session protocol** — automatically saves what you worked on for the next session

## How it works

```
/reflect writes → context-db.json stores → /standup reads and applies
```

The `/standup` skill reads your Jira backlog, git state, and context DB to generate a prioritized status report. It uses a simplified WSJF (Weighted Shortest Job First) scoring to rank what you should work on next, adjusted by patterns learned from your estimate history.

The `/reflect` skill lets you log decisions and observations mid-session. Over time, the context DB builds up patterns that make `/standup` recommendations smarter.

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- **Atlassian MCP** connected in Claude Code (for Jira integration)
- A Jira project (team-managed/next-gen)

### Jira label conventions

The kit uses labels for priority since team-managed Jira projects don't support the Priority field:

| Label | Meaning |
|-------|---------|
| `p0-blocker` | Must fix immediately — data loss, security, outage |
| `p1-critical` | Must fix soon — major feature broken |
| `p2-high` | Fix this week — important but not urgent |
| `p3-medium` | Future roadmap |

Additional labels: `quick-win`, `technical-debt`, `polish`, `silent-failure`, `data-corruption-risk`

## Installation

### 1. Copy the skills

Copy the `.claude/commands/` folder into your project:

```bash
cp -r claude-code-pm-kit/.claude/commands/ your-project/.claude/commands/
```

Or cherry-pick individual skills — they work independently.

### 2. Configure for your project

Each skill references `cloudId: "blueinit.atlassian.net"` and `projectKey: "CEN"`. Update these to match your Jira instance:

```bash
# Find your cloudId by connecting the Atlassian MCP in Claude Code,
# then asking Claude to call getAccessibleAtlassianResources
```

Replace in all skill files:
- `blueinit.atlassian.net` → your Jira site
- `CEN` → your project key

### 3. Set up the context DB

Create the context DB file:

```bash
mkdir -p .workspace-temp
echo '{"sessions": [], "estimates": [], "patterns": []}' > .workspace-temp/context-db.json
```

Add `.workspace-temp/` to your `.gitignore`.

### 4. Set up ADR folder (optional)

```bash
mkdir -p docs/decisions
```

Copy `docs/decisions/README.md` for the template and index.

### 5. Update CLAUDE.md (optional)

Add the end session protocol to your project's `CLAUDE.md` so the context DB gets updated automatically. See `CLAUDE.md.example` for the relevant sections.

## Customization

### Adjusting priority labels

Edit the label mappings in `standup.md` under "Auto-Scoring from Labels" and in `rfe.md` / `bug.md` under their respective label sections.

### Adding component labels

Add your project's component labels to `rfe.md` and `bug.md`. Examples: `frontend`, `backend`, `api`, `database`, `auth`, etc.

### Changing WSJF weights

The standup skill uses a simplified WSJF formula:

```
Score = (User Impact + Time Sensitivity + Risk Reduction) / Effort
```

Each factor is 1-5. Adjust the auto-scoring table in `standup.md` to match your label conventions.

## Architecture

```
┌─────────────┐     writes      ┌──────────────────┐
│  /reflect    │ ──────────────> │  context-db.json  │
└─────────────┘                 │  (.workspace-temp) │
                                └────────┬─────────┘
                                         │ reads
                                         v
┌─────────────┐     reads       ┌──────────────────┐
│  /standup    │ <────────────── │  Jira (MCP)       │
│              │ <────────────── │  Git state        │
│              │ <────────────── │  context-db.json  │
└─────────────┘                 └──────────────────┘

┌─────────────┐     creates     ┌──────────────────┐
│  /rfe        │ ──────────────> │  Jira Story       │
│  /bug        │ ──────────────> │  Jira Bug         │
│  /backlog    │ <────────────── │  Jira backlog     │
└─────────────┘                 └──────────────────┘

┌─────────────┐     permanent   ┌──────────────────┐
│  ADRs        │ ──────────────> │  docs/decisions/  │
└─────────────┘                 │  (git-tracked)    │
                                └──────────────────┘
```

## Design philosophy

- **Recommend, don't dictate** — presents priorities, you decide what to act on
- **Continuous flow** — no sprints, no ceremonies, priorities adjust daily
- **Data-driven** — WSJF scoring + pattern learning, not gut feel
- **Zero dependencies** — plain JSON file, no MCP servers, no databases
- **Own your schema** — everything is a markdown skill file you can edit

## License

MIT
