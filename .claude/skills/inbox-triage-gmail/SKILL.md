# Inbox Triage — Gmail

Scan Gmail for actionable emails and surface tasks in Todoist. Draft responses for manual sending.

---

## How to Invoke

```
/inbox-triage-gmail
/inbox-triage-gmail [number of hours to look back, default: 24]
```

---

## Prerequisites

- `gws` CLI installed and authenticated (`gws auth status` should show your account)
- Todoist MCP server connected (for creating tasks)

---

## ⚠️ Access Constraints

**Gmail access is READ-ONLY.** This is a hard rule.

| Action | Allowed? |
|---|---|
| Read emails | ✅ Yes |
| Search emails | ✅ Yes |
| Read labels | ✅ Yes |
| Draft a response (presented as text) | ✅ Yes — but never sent by the EA |
| Send an email | ❌ **Never** |
| Reply to an email | ❌ **Never** |
| Forward an email | ❌ **Never** |
| Delete an email | ❌ **Never** |
| Modify labels or archive | ❌ **Never** |

---

## Workflow

### Step 1: Fetch recent emails

```bash
gws gmail users messages list --params '{"userId": "me", "q": "is:unread in:inbox -category:promotions -category:social", "maxResults": 25}'
```

Then fetch metadata (subject, from, date) for each message ID:

```bash
gws gmail users messages get --params '{"userId": "me", "id": "MESSAGE_ID", "format": "metadata", "metadataHeaders": ["From", "Subject", "Date"]}'
```

For full body when needed:

```bash
gws gmail users messages get --params '{"userId": "me", "id": "MESSAGE_ID", "format": "full"}'
```

### Step 2: Assess each email

| Email Category | Action |
|---|---|
| **Requires action (<2 min)** | Flag for immediate handling. Show subject, sender, and suggested action. |
| **Requires action (>2 min)** | Create task in the appropriate Todoist project/section with due date. |
| **Needs a reply** | Draft a response and present it. Create a task: "Reply to [sender] re: [subject]". |
| **Waiting on someone** | Create task in Waiting For with context (who, what, when expected). Add `waiting` label. |
| **Reference / FYI** | Note it. Suggest filing in `references/` if worth saving. |
| **Not needed** | Skip. |

### Step 3: Cross-reference

- Check `context/current-priorities.md` for alignment with active priorities
- Check existing Todoist tasks to avoid creating duplicates
- Cross-reference `projects/` for relevant context

### Step 4: Choose the right Todoist project

Map emails to your project structure. Update this table to match your Todoist setup:

| If the email relates to... | Project | Section |
|---|---|---|
| [Your area 1] | [Project] | [Section] |
| [Your area 2] | [Project] | [Section] |
| Delegated / waiting on someone | Waiting For | — |

---

## Output

Display a summary table:

| From | Subject | Category | Action Taken | Task Created? |
|---|---|---|---|---|
| ... | ... | ... | ... | Yes / No |

If any drafted responses were created:

### Drafted Responses

**To:** [recipient]
**Re:** [subject]
**Draft:**
> [draft text]

*Copy and send manually.*

---

End with: "X emails scanned. Y tasks created. Z drafts prepared for your review."

---

## Rules

- **NEVER send, reply to, forward, or delete emails** — read-only access only
- Draft responses are suggestions — present them clearly for manual review and sending
- Don't create duplicate tasks — check existing Todoist tasks first
- Prioritize emails related to active priorities in `context/current-priorities.md`
- Skip newsletters, marketing, and obvious low-priority emails unless they contain action items
