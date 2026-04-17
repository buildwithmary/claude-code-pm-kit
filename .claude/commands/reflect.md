---
description: Log a decision, pattern, or estimate to the context DB for cross-session learning.
---

# Reflect — Log Context for Future Sessions

Record a decision, pattern observation, or estimate result to `.workspace-temp/context-db.json` so future sessions can learn from it.

## Input

$ARGUMENTS

## Usage

- `/reflect decision: chose X over Y because Z` — logs to current session's decisions
- `/reflect pattern: frontend work tends to take half the estimate` — logs to patterns array
- `/reflect estimate KEY-XXX estimated 2d actual 1d` — logs to estimates array

## Process

1. Read `.workspace-temp/context-db.json`
2. Based on the input type:
   - **decision**: Append to the most recent session's `decisions` array. If no session exists for today, create one.
   - **pattern**: Append to the top-level `patterns` array. Check for duplicates first.
   - **estimate**: Append to the `estimates` array with `ticket`, `estimated`, `actual`, and optional `note`.
3. Write the file back.
4. Confirm what was logged in one line.

## Rules

- Keep it brief — one-liners, not paragraphs
- Deduplicate patterns before adding
- Prune sessions to last 30 entries if over limit
- Don't ask clarifying questions — if the input is clear enough, just log it
