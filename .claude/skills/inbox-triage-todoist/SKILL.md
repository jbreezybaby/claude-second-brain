# Inbox Triage — Todoist

Process the Todoist Inbox using the GTD Clarify + Organize workflow. Goal: **Inbox zero.**

---

## How to Invoke

```
/inbox-triage-todoist
```

---

## Workflow

Pull all tasks from the Todoist Inbox project via MCP. **Triage all items first, present the full plan for approval, then execute all actions in one parallel batch.**

### Step 1: Fetch the inbox

Use `mcp__todoist__todoist_task_get` filtered to the Inbox project. Get the Inbox project ID via `mcp__todoist__todoist_project_get` if not known.

### Step 2: Is it actionable?

| Answer | Action |
|--------|--------|
| **No — trash** | Delete the task (`mcp__todoist__todoist_task_delete`) |
| **No — reference** | Note it, suggest filing in `references/` if worth saving. Delete the task. |
| **No — someday/maybe** | Move to Someday / Maybe project, add `someday` label |
| **Yes** | Continue to Step 3 |

### Step 3: What's the next action?

| Question | Action |
|----------|--------|
| **Takes < 2 minutes?** | Flag it, suggest doing it now. Add `quick-win` label. |
| **Waiting on someone else?** | Move to Waiting For project. Add `waiting` label. Add a comment with who/what/when. |
| **Multi-step project?** | Ensure a project folder exists in `projects/`. Create next single action in the relevant project/section. |
| **Single action, > 2 min** | Move to the appropriate project + section with a due date. |

### Step 4: Choose the right project and section

Map tasks to your Todoist projects and sections. Update this table to match your structure:

| If it relates to... | Project | Section |
|---------------------|---------|---------|
| [Your area 1] | [Project name] | [Section name] |
| [Your area 2] | [Project name] | [Section name] |
| Delegated / blocked | Leave in current project | Add `waiting` label |
| No urgency, future idea | Leave in current project | Add `someday` label |

### Step 5: Add metadata

When placing a task:
- Set **due date** if one is apparent or can be inferred
- Set **priority** (p1–p4) based on urgency and alignment with `context/current-priorities.md`
  - p1 = urgent, p2 = high, p3 = medium, p4 = low
- Add **labels** as appropriate: `urgent`, `quick-win`, `waiting`, `someday`

---

## Project & Section IDs

Get live project and section IDs via MCP rather than hardcoding them:

```
mcp__todoist__todoist_project_get        # list all projects
mcp__todoist__todoist_section_get        # list sections for a given project_id
```

---

## Output

After processing, display a summary table:

| Item | Action Taken | Destination | Section | Due Date | Labels |
|------|-------------|-------------|---------|----------|--------|
| ... | Moved / Deleted / Flagged | ... | ... | ... | ... |

End with: "Inbox: X items processed, Y remaining."

---

## Rules

- Always process ALL items — don't leave anything in Inbox
- When in doubt about categorization, ask the user
- Reference `context/current-priorities.md` and `context/goals.md` to inform priority decisions
- If an item implies a new project that doesn't exist in `projects/`, flag it before creating
- **Plan first, execute in one batch** — present the full triage plan for approval, then fire all moves/updates in a single parallel batch

## Key MCP Tools

| Action | Tool |
|--------|------|
| Get inbox tasks | `mcp__todoist__todoist_task_get` |
| Move task to project/section | `mcp__todoist__todoist_task_update` (use `project_id` + `section_id`) |
| Update task (due date, priority, labels) | `mcp__todoist__todoist_task_update` |
| Delete task | `mcp__todoist__todoist_task_delete` |
| Add comment | `mcp__todoist__todoist_comment_create` |
| Create new task | `mcp__todoist__todoist_task_create` |
