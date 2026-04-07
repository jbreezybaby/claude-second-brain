# Weekly Review

The GTD weekly review — a full system scan to ensure nothing is slipping and everything is aligned. This is the heartbeat of the system.

---

## How to Invoke

```
/weekly-review
```

---

## Prerequisites

- Todoist MCP server connected
- `gws` CLI installed and authenticated (for calendar review)

---

## Workflow

Run through all 10 steps in order. Present findings after each section.

### Step 1: Clear Inbox
Process any remaining items in the Todoist Inbox (runs the `/inbox-triage-todoist` workflow).

### Step 2: Review Next Actions
Pull all incomplete tasks from each Todoist project. For each, display:

| Task | Due Date | Priority | Status |
|---|---|---|---|
| ... | ... | ... | Overdue / Due this week / Upcoming |

Flag overdue items. For each: keep, reschedule, or complete?

### Step 3: Review Waiting For
Pull all tasks with the `waiting` label.

| Waiting For | Who | Since | Follow-up Needed? |
|---|---|---|---|
| ... | ... | ... | Yes/No |

For items waiting too long, suggest a follow-up action.

### Step 4: Review Projects
For each active project in `projects/`:
- Read the project README
- Does it have at least one clear next action in Todoist?
- Is the README status still current?
- Any blockers?

| Project | Next Action Exists? | Status Current? | Notes |
|---|---|---|---|
| ... | Yes/No | Yes/No | ... |

### Step 5: Review Areas
Read `context/areas.md`. For each Area of Responsibility:
- Is this area being neglected?
- Any actions needed?

| Area | Last Activity | Needs Attention? |
|---|---|---|
| ... | ... | Yes/No |

### Step 6: Review Someday/Maybe
Pull all tasks with the `someday` label.
- Anything ready to activate? Move to the appropriate project/section.
- Anything no longer relevant? Delete.

### Step 7: Review Calendar
Scan the next 7-14 days of Google Calendar via `gws`.
- Any events that need prep tasks?
- Any conflicts or scheduling issues?
- Cross-reference key dates from `context/current-priorities.md`

### Step 8: Review Goals
Read `context/goals.md`.
- Are current tasks and projects aligned with quarterly goals?
- Any goal being neglected?

| Goal | Aligned Tasks/Projects | On Track? |
|---|---|---|
| ... | ... | Yes / At Risk / Off Track |

### Step 9: Surface New Tasks
Based on the full review, suggest tasks that may not have been captured:
- Follow-ups that should exist but don't
- Prep work for upcoming events
- Actions implied by project status

### Step 10: Update Priorities
If priorities have shifted based on the review:
- Suggest updates to `context/current-priorities.md`
- Ask for confirmation before making changes

### Step 11: Sync Public Repo
Run `/sync-public-repo` to push any shareable skill or system changes made since the last sync.

---

## Output

End with a weekly summary:

| Category | Count |
|---|---|
| Inbox items processed | X |
| Overdue tasks | X |
| Tasks due this week | X |
| Waiting For items | X |
| Projects reviewed | X |
| New tasks created | X |

**Top 3 priorities for the coming week:**
1. ...
2. ...
3. ...

---

## Rules

- This is an interactive review — pause after each section for input
- Don't rush through it — the weekly review is where the system maintains trust
- If the user says "skip" on a section, skip it
- Update project READMEs if status has changed
- Log any significant decisions in `decisions/log.md`
