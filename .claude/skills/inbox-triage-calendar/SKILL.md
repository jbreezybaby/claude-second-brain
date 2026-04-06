# Inbox Triage — Google Calendar

Scan upcoming Google Calendar events and surface preparation tasks in Todoist.

---

## How to Invoke

```
/inbox-triage-calendar
/inbox-triage-calendar [number of days to look ahead, default: 7]
```

---

## Prerequisites

- `gws` CLI installed and authenticated (`gws auth status` should show your account)
- Todoist MCP server connected (for creating tasks)

---

## Workflow

### Step 1: Fetch upcoming events

Pull events from Google Calendar via `gws` CLI for the next 7 days (or specified range).

```bash
gws calendar events list --params '{"calendarId": "primary", "timeMin": "TODAY_ISO", "timeMax": "7_DAYS_ISO", "singleEvents": true, "orderBy": "startTime", "maxResults": 50}'
```

Replace `TODAY_ISO` and `7_DAYS_ISO` with RFC3339 timestamps (e.g. `2026-04-06T00:00:00-04:00`).

### Step 2: Assess each event

| Event Type | Action |
|---|---|
| **Meeting/call with prep needed** | Create task: "Prep for [meeting] — [suggested prep]". Set due date to day before. |
| **Interview** | Create prep tasks. Check `projects/` for existing research. Set due date to day before. |
| **Appointment** | Create task with logistics (what to bring, arrive by time). Set due date to day of event. |
| **Deadline / due date** | Create task in Todoist if one doesn't already exist. |
| **Recurring event with no action needed** | Skip. |
| **Social event** | Create task only if logistics needed (gift, RSVP, directions). |

### Step 3: Cross-reference

- Check `context/current-priorities.md` for key dates that may not be on the calendar
- Check existing Todoist tasks to avoid creating duplicates
- Cross-reference `projects/` for relevant context

### Step 4: Choose the right Todoist project

Map events to your project structure. Update this table to match your Todoist setup:

| If the event relates to... | Project | Section |
|---|---|---|
| [Your area 1] | [Project] | [Section] |
| [Your area 2] | [Project] | [Section] |

---

## Output

Display a summary table:

| Date | Event | Prep Task Created | Project | Due |
|---|---|---|---|---|
| ... | ... | ... | ... | ... |

End with: "X events scanned. Y prep tasks created."

---

## Rules

- Don't create duplicate tasks — check existing Todoist tasks first
- Set prep task due dates to the day BEFORE the event, not the day of
- When a calendar event implies multi-step preparation, create separate tasks for each step
- Skip recurring events that clearly need no preparation
