# Schedule Day

Build a structured daily schedule and write it to Google Calendar as time-blocked events.

---

## How to Invoke

```
/schedule-day [today|tomorrow|YYYY-MM-DD]
```

Default: today.

---

## Prerequisites

- `gws` CLI authenticated
- Todoist MCP connected
- An "EA Schedule" calendar exists in Google Calendar (create one and hardcode its ID below)

---

## Calendar IDs (configure before use)

| Calendar | ID | Role |
|----------|----|------|
| EA Schedule | `[YOUR_EA_SCHEDULE_CALENDAR_ID]` | Write target — all skill events go here |
| Primary | `[YOUR_PRIMARY_CALENDAR_EMAIL]` | Conflict source |
| Other calendars | `[YOUR_OTHER_CALENDAR_IDS]` | Conflict source |

---

## Standing Schedule (always created)

Customize these to match your daily routine:

| Time | Block |
|------|-------|
| 9:30–9:45 | Stretching & Reading |
| 9:45–10:00 | EA Morning Briefing |
| 12:00–12:30 | Exercise |
| 12:30–1:30 | Midday Break |

Open work windows: **10:00–12:00** and **1:30–5:00** (skill fills these in).

---

## Workflow

### Step 1: Resolve Target Date

Parse the argument:
- `today` or no argument → today's date
- `tomorrow` → tomorrow's date
- `YYYY-MM-DD` → that date

Determine the UTC offset for your timezone on that date. Use throughout all API calls.

---

### Step 2: Cleanup Previous Skill Events

Fetch all events on the EA Schedule calendar for the target day:
```bash
gws calendar events list --params '{
  "calendarId": "[YOUR_EA_SCHEDULE_CALENDAR_ID]",
  "timeMin": "DATE_T00:00:00OFFSET",
  "timeMax": "DATE_T23:59:59OFFSET",
  "singleEvents": true
}' 2>&1
```

Delete any events whose `description` contains `#ea-scheduled`:
```bash
gws calendar events delete --params '{
  "calendarId": "[YOUR_EA_SCHEDULE_CALENDAR_ID]",
  "eventId": "EVENT_ID"
}' 2>&1
```

**Never delete events without `#ea-scheduled` in the description.**

---

### Step 3: Fetch Existing Events (Conflict Detection + Context)

Fetch the target day's events from all relevant calendars in parallel:
```bash
gws calendar events list --params '{
  "calendarId": "CALENDAR_ID",
  "timeMin": "DATE_T00:00:00OFFSET",
  "timeMax": "DATE_T23:59:59OFFSET",
  "singleEvents": true,
  "orderBy": "startTime"
}' 2>&1
```

Classify results:
- **Conflicts**: timed events that overlap the open work windows (10:00–12:00 or 1:30–5:00)
- **Context events**: all-day events — not conflicts, but inform priorities

---

### Step 4: Fetch Context

Run in parallel:

**Todoist:**
```
todoist_task_get(filter="today")   # due today
todoist_task_get(filter="overdue") # overdue
```
Group by project/area. Sort by priority (p1 → p4).

**Priorities file:**
Read `context/current-priorities.md` — identifies the top strategic areas to weight heavily.

**Gmail (if not already retrieved this session):**
```bash
gws gmail users messages list --params '{
  "userId": "me",
  "q": "is:unread in:inbox -category:promotions -category:social",
  "maxResults": 10
}' 2>&1
```
Fetch metadata (From, Subject) for each result. Flag high-urgency items.

---

### Step 5: Build the Schedule

**Place standing blocks** (customize for your routine).

**Determine free windows:**
Start with the open windows. Subtract any conflicts from Step 3 to get a list of free sub-windows. Any sub-window shorter than 30 minutes is skipped.

**Theme priority order** (derived from `current-priorities.md` + Todoist + Gmail):
Define your own themes based on your active projects and areas. Suggested structure:
1. Most urgent active project
2. Second priority project
3. ...
4. Admin / Wrap-up (catch-all for misc overdue tasks, short items)
5. Learning block — schedule a 90-min block if the afternoon window has capacity

**Block-filling rules:**
- Minimum block size: **30 minutes**
- Buffer between consecutive work blocks: **15 minutes**
- Each theme gets at most one block per session (morning or afternoon), unless it's the dominant priority
- Block `summary` = theme name (e.g., "Job Search")
- Block `description` = bullet list of specific tasks within that theme + `\n\n#ea-scheduled`
- If no specific tasks exist for a theme, use a generic description
- Group related items together

**Heuristic:**
- Morning window: highest 1–2 priority themes
- Afternoon window: remaining themes + admin

---

### Step 6: Preview (wait for approval before creating)

Output three sections:

#### Part A — Day Summary
2–4 sentences contextualizing what the schedule accomplishes.

#### Part B — Not Scheduled Today
Items that exist but weren't given a block.

| Item | Reason Not Scheduled |
|------|---------------------|
| ... | Day full — consider tomorrow |

#### Part C — Proposed Schedule

| Time | Block | Details |
|------|-------|---------|
| 9:30–9:45 | Stretching & Reading | Standing block |
| ... | ... | ... |

Then ask: **"Create these events?"** — wait for explicit approval before proceeding to Step 7.

---

### Step 7: Create Events

Create each block sequentially. For each block:

```bash
gws calendar events insert --params '{
  "calendarId": "[YOUR_EA_SCHEDULE_CALENDAR_ID]"
}' --json '{
  "summary": "BLOCK_TITLE",
  "description": "TASK_DETAILS\n\n#ea-scheduled",
  "start": {"dateTime": "DATE_TSTART_TIMEOFFSET", "timeZone": "[YOUR_TIMEZONE]"},
  "end": {"dateTime": "DATE_TEND_TIMEOFFSET", "timeZone": "[YOUR_TIMEZONE]"}
}' 2>/dev/null
echo "exit: $?"
```

**Critical rules to prevent duplication:**
- Always use `2>/dev/null` — the gws CLI prints a keyring header to stderr
- **Check exit code (`$?`), not JSON output, to determine success.** Exit 0 = event created.
- **Never retry a create command within a single run.** If exit code was 0, the event exists — retrying will duplicate it.
- Run each `gws events insert` call individually and sequentially

---

### Step 8: Confirm

Display a compact table of what was created:

| Time | Block | Status |
|------|-------|--------|
| 9:30–9:45 | Stretching & Reading | Created |
| ... | ... | ... |

---

## Rules

- **Never** delete or modify calendar events not created by this skill
- All skill-created events must include `#ea-scheduled` in the description
- Standing blocks are always created — never skipped or moved
- Minimum block: 30 minutes. Skip free windows under 30 minutes.
- 15-minute buffer between all consecutive work blocks
- Always preview and wait for approval before writing to calendar
- If `/morning-briefing` was already run this session, reuse its context — don't re-fetch Gmail or Todoist
