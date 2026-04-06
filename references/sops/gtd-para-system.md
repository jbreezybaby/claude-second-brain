# GTD + PARA Task Management System

*Design reference for the EA's task management architecture.*

---

## Core Principle

**Todoist = single source of truth for all tasks.** The EA is a processor and planner, not a task store. All inboxes (Gmail, Todoist Inbox, Google Calendar) get processed into Todoist.

---

## GTD Workflow

| Step | Tool | How It Works |
|---|---|---|
| **Capture** | Todoist Inbox + Gmail + Google Calendar | Todoist app/widget, email scanning, calendar event scanning |
| **Clarify** | `/inbox-triage-*` skills | Process each item: actionable? next action? <2 min? |
| **Organize** | Todoist projects + sections (mapped to PARA) | Items move to the right project/section |
| **Reflect** | `/weekly-review` + `/morning-briefing` | Weekly: full system review. Daily: priority scan |
| **Engage** | Prioritized task list | Work from the list with confidence |

---

## Todoist Structure

The EA queries Todoist live via MCP for current project and section structure — no static copy is maintained here to avoid drift.

To inspect the current structure: `mcp__todoist__todoist_project_get` (all projects) or `mcp__todoist__todoist_section_get` (sections within a project).

Suggested project structure (adapt to your life):

| Project | Purpose |
|---------|---------|
| Inbox | Capture point — process to zero regularly |
| [Area 1] | e.g., Work, Career, Side Business |
| [Area 2] | e.g., Home, Family |
| Waiting For | Delegated items and external dependencies |
| Someday / Maybe | Ideas and future possibilities |

---

## PARA Structure

| PARA | EA Folder | What Lives Here |
|---|---|---|
| **Projects** | `projects/` | Active workstreams with deadlines |
| **Areas** | `context/` | Ongoing responsibilities (see `context/areas.md`) |
| **Resources** | `references/` | Information for later |
| **Archives** | `archives/` | Completed or inactive material |

---

## Skills

| Skill | Purpose | Frequency |
|---|---|---|
| `/inbox-triage-todoist` | Process Todoist Inbox → organize | As needed |
| `/inbox-triage-gmail` | Scan Gmail (read-only) → create tasks in Todoist | Daily |
| `/inbox-triage-calendar` | Scan calendar → create prep tasks in Todoist | Daily / weekly |
| `/weekly-review` | Full GTD system review (10 steps) | Weekly |
| `/morning-briefing` | Daily orientation and focus | Weekday mornings |

---

## Gmail Access Policy

The EA has **read-only** Gmail access. It can read and search emails but **never** send, reply, forward, or delete. When a response is needed, the EA drafts it and the user sends manually.

---

## Flow

```
CAPTURE                     CLARIFY + ORGANIZE              REFLECT           ENGAGE
─────────────────           ──────────────────              ───────           ──────
Todoist app / widget   →  /inbox-triage-todoist
Gmail (read-only)      →  /inbox-triage-gmail        →    /weekly-review  →  /morning-briefing
Google Calendar        →  /inbox-triage-calendar           (weekly)           (daily)
```
