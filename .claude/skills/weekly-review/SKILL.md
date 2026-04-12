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
- Google Workspace MCP server connected (for calendar review)

---

## Workflow

Run through all 10 steps in order. Present findings after each section.

### Step 1: Clear Inbox
Process any remaining items in the Todoist Inbox (runs the `/inbox-triage-todoist` workflow).

### Step 2: Review Next Actions
Pull all incomplete tasks from Todoist (all projects). Group by project.

For each project, display:

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

### Step 4: Review Projects & Areas

**Definitions:**
- **Project** — an active commitment with a finish line. When complete, it gets archived. Lives in `projects/`.
- **Area** — an ongoing life domain maintained indefinitely. No finish line. Projects often live *within* areas (e.g., Job Search is a project inside Career).

The goal here is not to enumerate tasks — Step 2 did that. This step is about pulling back and thinking at the level of commitments and life domains. Read the README for each project and `context/areas.md` for areas, then synthesize.

**For each active project**, present a brief narrative covering:
- **Purpose** — why does this project exist? What's the outcome we're working toward?
- **Current position** — where does it stand right now, in plain English? What's the overall momentum?
- **Priority** — relative to everything else, how urgent/important is this right now?
- **Big-picture question** — is this still the right project to have open? Or has the situation changed?

Keep it to 3–5 sentences per project. Don't list tasks — synthesize.

**For each area**, present a brief narrative covering:
- **Purpose** — what does this area represent in your life?
- **Health** — is it getting appropriate attention? Any key indicators slipping?
- **Active projects** — which open projects fall within this area? Any gaps?
- **Big-picture question** — is this area being neglected? Does it need a project spun up?

Keep it to 2–4 sentences per area.

**Priority alignment check** — after presenting all projects and areas, ask: *"Based on all of this, are the right things getting your attention right now? Anything you want to shift focus toward or away from?"*

### Step 5: Review Backlog & New Ideas

**Definition:** Someday/Maybe = things you're *interested in* but haven't committed to. Not a current project or responsibility — just "maybe one day."

Pull all tasks from the Todoist Someday / Maybe project.
- Anything ready to activate? → Promote to an active project.
- Anything stale or no longer interesting? → Delete.
- Any new ideas from the past week that should land here rather than disappear?

### Step 6: Review Calendar
Scan the next 7-14 days of Google Calendar.
- Any events that need prep tasks?
- Any conflicts or scheduling issues?
- Cross-reference key dates from `context/current-priorities.md`

### Step 7: Review Goals
Read `context/goals.md`.
- Are current tasks and projects aligned with quarterly goals?
- Any goal being neglected?

| Goal | Aligned Tasks/Projects | On Track? |
|---|---|---|
| ... | ... | Yes / At Risk / Off Track |

### Step 8: Surface New Tasks
Based on the full review, suggest tasks that may not have been captured:
- Follow-ups that should exist but don't
- Prep work for upcoming events
- Actions implied by project status

### Step 9: Update Priorities
If priorities have shifted based on the review:
- Suggest updates to `context/current-priorities.md`
- Ask for confirmation before making changes

### Step 10: Sync Public Repo
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
