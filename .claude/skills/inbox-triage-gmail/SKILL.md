# Inbox Triage — Gmail

Apply project and priority labels to unread Gmail emails that haven't been triaged yet. Runs autonomously — no approval step. Emails remain **unread** after labeling.

Processes 100 emails per run, newest first. If a backlog exists, run again to continue.

---

## How to Invoke

```
/inbox-triage-gmail
```

---

## Access Constraints

| Action | Allowed? |
|---|---|
| Read emails | ✅ Yes |
| Search emails | ✅ Yes |
| Apply / modify labels | ✅ Yes — labels only |
| Mark as read | ❌ Never — all emails must stay unread |
| Send, reply, forward | ❌ Never |
| Delete emails | ❌ Never |
| Archive emails | ❌ Never |

---

## Label Taxonomy

Labels are pre-created in Gmail. Look up your label IDs using:

```bash
gws gmail users labels list --params '{"userId":"me"}'
```

Then fill in this table with your label IDs:

### Project Labels

| Label Name | ID |
|---|---|
| `[Project 1]` | `Label_XXX` |
| `[Project 2]` | `Label_XXX` |
| `[Project 3]` | `Label_XXX` |
| `[Project 4]` | `Label_XXX` |

### Priority Labels

| Label Name | ID | Meaning |
|---|---|---|
| `P1` | `Label_XXX` | Act today — time-sensitive |
| `P2` | `Label_XXX` | Act this week — important |
| `P3` | `Label_XXX` | Useful, no hard deadline |
| `P4` | `Label_XXX` | FYI, receipts, notifications |

**All triage label IDs** — list all of them for use in `TRIAGE_LABELS` in the scripts below.

---

## Workflow

The entire skill runs in **3 tool calls total**: one Bash to fetch, one Write to record classifications, one Bash to apply labels.

### Step 1: Fetch untriaged emails (1 Bash call)

Run this Python script as a single Bash call. It fetches up to 500 unread message IDs, checks each for triage labels using 20 parallel threads, collects the first 100 untriaged ones, and saves to `/tmp/gmail_untriaged.json`.

```python
python3 << 'EOF'
import subprocess, json, sys
from concurrent.futures import ThreadPoolExecutor, as_completed

# Replace with all your triage label IDs
TRIAGE_LABELS = {'Label_XXX','Label_XXX','Label_XXX','Label_XXX','Label_XXX','Label_XXX','Label_XXX','Label_XXX'}
MAX_WORKERS = 20
TARGET = 100

def get_metadata(mid):
    result = subprocess.run(
        ['gws', 'gmail', 'users', 'messages', 'get', '--params',
         f'{{"userId":"me","id":"{mid}","format":"metadata","metadataHeaders":["From","Subject","Date"]}}'],
        capture_output=True, text=True
    )
    out = result.stdout
    start = out.find('{')
    if start < 0:
        return None
    try:
        j = json.loads(out[start:])
        labels = set(j.get('labelIds', []))
        headers = {h['name']: h['value'] for h in j.get('payload', {}).get('headers', [])}
        return {
            'id': j['id'],
            'has_triage': bool(TRIAGE_LABELS & labels),
            'from': headers.get('From', ''),
            'subject': headers.get('Subject', ''),
            'date': headers.get('Date', '')
        }
    except:
        return None

# Fetch unread message IDs
r = subprocess.run(
    ['gws', 'gmail', 'users', 'messages', 'list', '--params',
     '{"userId":"me","q":"is:unread","maxResults":500}'],
    capture_output=True, text=True
)
out = r.stdout
start = out.find('{')
data = json.loads(out[start:])
all_ids = [m['id'] for m in data.get('messages', [])]
print(f"Fetched {len(all_ids)} unread message IDs", flush=True)

# Fetch metadata in parallel, preserving order (newest first), stop at TARGET untriaged
untriaged = []
inspected = 0
batch_size = 50

for batch_start in range(0, len(all_ids), batch_size):
    if len(untriaged) >= TARGET:
        break
    batch = all_ids[batch_start:batch_start + batch_size]

    results = {}
    with ThreadPoolExecutor(max_workers=MAX_WORKERS) as ex:
        futures = {ex.submit(get_metadata, mid): mid for mid in batch}
        for f in as_completed(futures):
            mid = futures[f]
            results[mid] = f.result()

    for mid in batch:
        if len(untriaged) >= TARGET:
            break
        inspected += 1
        meta = results.get(mid)
        if meta and not meta['has_triage']:
            untriaged.append(meta)

print(f"Inspected {inspected} | Already triaged: {inspected - len(untriaged)} | Found {len(untriaged)} untriaged")

with open('/tmp/gmail_untriaged.json', 'w') as f:
    json.dump(untriaged, f, indent=2)
print("Saved to /tmp/gmail_untriaged.json")
EOF
```

### Step 2: Read the file and classify (no Bash call)

Read `/tmp/gmail_untriaged.json` with the Read tool. Classify each email:

**Project classification** (customize to your projects):

| If it relates to... | Label |
|---|---|
| Job applications, interviews, recruiters, offers | Job Search label |
| [Your properties or real estate] | Properties label |
| [Your business] | Business label |
| Family, kids, household, health, personal finance | Family label |

**Priority classification:**

| Signal | Priority |
|---|---|
| Interview scheduled, offer received, urgent recruiter response needed | P1 |
| Job application confirmation, recruiter outreach, legal deadlines, closing docs | P2 |
| Business updates (non-urgent), networking, utilities, property updates | P3 |
| Receipts, order confirmations, newsletters, account notifications | P4 |

Key rules:
- Job search always skews high — P2 minimum for any real human contact
- Time-sensitive beats everything — deadline today or tomorrow = P1
- When in doubt between two priority levels, go higher
- Newsletters and automated notifications: P4 unless they contain action items

Build a classification list in this format and write it to `/tmp/gmail_labels.json` using the Write tool:

```json
[
  {"id": "MESSAGE_ID", "priority": "Label_XXX", "project": "Label_XXX"},
  {"id": "MESSAGE_ID", "priority": "Label_XXX", "project": null}
]
```

`project` is `null` if no project label applies.

### Step 3: Apply labels (1 Bash call)

```python
python3 << 'EOF'
import subprocess, json
from concurrent.futures import ThreadPoolExecutor, as_completed

with open('/tmp/gmail_labels.json') as f:
    classifications = json.load(f)

def apply_labels(item):
    labels = [item['priority']]
    if item.get('project'):
        labels.append(item['project'])
    payload = json.dumps({"addLabelIds": labels})
    result = subprocess.run(
        ['gws', 'gmail', 'users', 'messages', 'modify',
         '--params', f'{{"userId":"me","id":"{item["id"]}"}}',
         '--json', payload],
        capture_output=True, text=True
    )
    return item['id'], result.returncode == 0

ok = 0
errors = 0
with ThreadPoolExecutor(max_workers=20) as ex:
    futures = [ex.submit(apply_labels, item) for item in classifications]
    for f in as_completed(futures):
        mid, success = f.result()
        if success:
            ok += 1
        else:
            errors += 1
            print(f"ERROR: {mid}")

print(f"Done: {ok} labeled, {errors} errors")
EOF
```

### Step 4: Output summary

Display the results table:

| From | Subject | Project | Priority |
|---|---|---|---|
| ... | ... | [Project] | P2 |

End with one of:
- If fully caught up: "X emails inspected. Y already triaged. Z labeled. Inbox fully triaged."
- If backlog remains: "X emails inspected. Y already triaged. Z labeled. Run `/inbox-triage-gmail` again to continue."

---

## Rules

- Emails must remain **unread** — never add `UNREAD` to `removeLabelIds`
- Always assign a priority label — never leave an untriaged email without one
- Project label is optional — not every email maps to a project
- Skip any email that already has one or more of the triage labels
- Process newest emails first
- For job-related emails from real humans, default to P2 or higher
- Do not create Todoist tasks — labeling is the only output of this skill
- Do not send, reply, forward, delete, or archive any email
