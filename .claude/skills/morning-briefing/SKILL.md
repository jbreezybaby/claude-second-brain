# Morning Briefing

Quick daily orientation — what to focus on today. Short, scannable, action-oriented.

---

## How to Invoke

```
/morning-briefing
```

---

## Prerequisites

- Todoist MCP server connected
- `gws` CLI installed and authenticated (for calendar and email via Bash tool)

---

## Workflow

### 1. Today's Calendar

Pull today's events from Google Calendar via `gws`:

```bash
gws calendar events list --params '{"calendarId": "primary", "timeMin": "TODAY_T00:00:00-04:00", "timeMax": "TODAY_T23:59:59-04:00", "singleEvents": true, "orderBy": "startTime"}'
```

| Time | Event | Prep Needed? |
|---|---|---|
| ... | ... | Yes/No |

### 2. Today's Tasks

Pull tasks due today from Todoist (filter: `today`). Sort by priority.

| Task | Project | Priority |
|---|---|---|
| ... | ... | High/Med/Low |

### 3. Overdue

Pull overdue tasks from Todoist (filter: `overdue`). These need to be dealt with or rescheduled.

| Task | Project | Was Due | Action Needed |
|---|---|---|---|
| ... | ... | ... | Complete / Reschedule / Delete |

### 4. Key Dates This Week

Cross-reference `context/current-priorities.md` and Google Calendar for upcoming deadlines.

| Date | What | Days Away |
|---|---|---|
| ... | ... | ... |

### 5. Waiting For Check

Pull tasks with the `waiting` label from Todoist that should have come back by now.

| Item | Waiting On | Since | Overdue? |
|---|---|---|---|
| ... | ... | ... | Yes/No |

### 6. Email Highlights

Quick scan of Gmail for urgent or important unread emails via `gws` (not a full triage — just flagging).

```bash
gws gmail users messages list --params '{"userId": "me", "q": "is:unread in:inbox -category:promotions -category:social", "maxResults": 10}'
```

Fetch metadata for each result, then surface only High urgency items.

| From | Subject | Urgency |
|---|---|---|
| ... | ... | High/Normal |

---

## Output Format

Keep the entire briefing concise — aim for one screen of output. Use tables throughout. No lengthy paragraphs.

End with:

**Today's focus:** [1-2 sentence recommendation of what to prioritize today based on the above]

---

## Rules

- Be concise — this is a morning scan, not a deep review
- Don't process the full inbox — just flag urgent emails
- If there are no overdue items, skip that section
- If there are no waiting-for items due, skip that section
- Always end with a clear "today's focus" recommendation
- Reference `context/current-priorities.md` to align focus with strategic priorities
