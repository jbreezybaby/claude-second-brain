# Job Scan

Scan LinkedIn job alert emails from the last 48 hours, open each job posting link to read the full description, score each role against James's profile, and create Todoist tasks for the best matches.

Runs autonomously — no approval step. Tasks appear in **Todoist > Job Search > Prep & Research**.

---

## How to Invoke

```
/job-scan
```

---

## Todoist IDs

| | ID |
|---|---|
| Job Search project | `[YOUR_JOB_SEARCH_PROJECT_ID]` |
| Prep & Research section | `[YOUR_PREP_RESEARCH_SECTION_ID]` |

---

## State File

`projects/job-search/job-scan-state.json`

```json
{
  "seen_ids": ["1234567890", "9876543210"],
  "last_run": "2026-04-14"
}
```

- `seen_ids` — LinkedIn job IDs already processed (regardless of fit score). Prevents re-surfacing on future runs.
- `last_run` — ISO date of most recent run.

If the file doesn't exist, create it with `{"seen_ids": [], "last_run": null}` before proceeding.

---

## Workflow

### Step 1 — Fetch LinkedIn Alert Emails

Search Gmail for LinkedIn job alert emails from the last 48 hours and get their message IDs:

LinkedIn sends job alert emails from two different addresses — both must be captured:

| Sender | Email type |
|--------|-----------|
| `jobs-noreply@linkedin.com` | Saved jobs reminders and job digest emails |
| `jobalerts-noreply@linkedin.com` | Individual job alert emails (e.g., "Staff PM at Acme: $185K/year") |

The `-label:job-scan` filter skips emails already processed in a prior run, making the search self-deduplicating at the email level (in addition to the job-ID-level dedup in Step 3).

This single script searches Gmail, collects message IDs, and fetches all full email bodies. Message IDs are saved to `/tmp/linkedin_message_ids.json` for use in Step 8.

```python
python3 << 'EOF'
import subprocess, json, base64

# Search for unprocessed LinkedIn job emails
search_result = subprocess.run(
    ['gws', 'gmail', 'users', 'messages', 'list', '--params',
     '{"userId":"me","q":"from:(jobs-noreply@linkedin.com OR jobalerts-noreply@linkedin.com) newer_than:2d -label:job-scan","maxResults":30}'],
    capture_output=True, text=True
)
out = search_result.stdout
start = out.find('{')
if start < 0:
    print("No emails found or Gmail error")
    json.dump([], open('/tmp/linkedin_emails.json', 'w'))
    json.dump([], open('/tmp/linkedin_message_ids.json', 'w'))
    exit()

data = json.loads(out[start:])
message_ids = [m['id'] for m in data.get('messages', [])]
print(f"Found {len(message_ids)} unprocessed LinkedIn emails")

# Save IDs for Step 8 (labeling)
with open('/tmp/linkedin_message_ids.json', 'w') as f:
    json.dump(message_ids, f)

def get_full_body(mid):
    result = subprocess.run(
        ['gws', 'gmail', 'users', 'messages', 'get', '--params',
         f'{{"userId":"me","id":"{mid}","format":"full"}}'],
        capture_output=True, text=True
    )
    out = result.stdout
    start = out.find('{')
    if start < 0:
        return None
    try:
        j = json.loads(out[start:])
        payload = j.get('payload', {})

        def extract_html(p):
            if p.get('mimeType') == 'text/html':
                data = p.get('body', {}).get('data', '')
                if data:
                    return base64.urlsafe_b64decode(data + '==').decode('utf-8', errors='replace')
            for part in p.get('parts', []):
                r = extract_html(part)
                if r:
                    return r
            return ''

        return {'id': mid, 'html': extract_html(payload)}
    except:
        return None

results = [d for mid in message_ids if (d := get_full_body(mid))]
with open('/tmp/linkedin_emails.json', 'w') as f:
    json.dump(results, f)
print(f"Fetched {len(results)} email bodies")
EOF
```

### Step 2 — Parse Job Listings from Emails

Use Python to extract job listings from the HTML email bodies. LinkedIn alert emails contain job links in the form `https://www.linkedin.com/comm/jobs/view/NNNNNNNN` (with tracking params after). Extract the job ID from each unique URL. Note: some email types use `/jobs/view/` and others use `/comm/jobs/view/` — the regex handles both.

```python
python3 << 'EOF'
import json, re
from html.parser import HTMLParser

class JobParser(HTMLParser):
    def __init__(self):
        super().__init__()
        self.jobs = {}
        self.current_url = None
        self.current_job_id = None
        self.capture_text = False
        self.current_text = []

    def handle_starttag(self, tag, attrs):
        if tag == 'a':
            attrs_dict = dict(attrs)
            href = attrs_dict.get('href', '')
            # Match LinkedIn job URLs
            m = re.search(r'linkedin\.com/(?:comm/)?jobs/view/(\d+)', href)
            if m:
                self.current_job_id = m.group(1)
                # Clean URL — strip tracking params
                self.current_url = f"https://www.linkedin.com/jobs/view/{self.current_job_id}"
                self.capture_text = True
                self.current_text = []

    def handle_endtag(self, tag):
        if tag == 'a' and self.current_job_id:
            text = ' '.join(self.current_text).strip()
            if text and self.current_job_id not in self.jobs:
                self.jobs[self.current_job_id] = {
                    'job_id': self.current_job_id,
                    'url': self.current_url,
                    'title_from_email': text,
                    'company_from_email': '',
                    'location_from_email': ''
                }
            self.current_job_id = None
            self.capture_text = False

    def handle_data(self, data):
        if self.capture_text:
            text = data.strip()
            if text:
                self.current_text.append(text)

with open('/tmp/linkedin_emails.json') as f:
    emails = json.load(f)

all_jobs = {}
for email in emails:
    parser = JobParser()
    parser.feed(email.get('html', ''))
    all_jobs.update(parser.jobs)

jobs = list(all_jobs.values())
with open('/tmp/linkedin_jobs_raw.json', 'w') as f:
    json.dump(jobs, f, indent=2)
print(f"Extracted {len(jobs)} unique job listings")
for j in jobs:
    print(f"  [{j['job_id']}] {j['title_from_email']}")
EOF
```

### Step 3 — Deduplicate

Load the state file and filter out already-seen jobs:

```python
python3 << 'EOF'
import json, os, glob

# Load state
state_path = 'projects/job-search/job-scan-state.json'
if os.path.exists(state_path):
    with open(state_path) as f:
        state = json.load(f)
else:
    state = {"seen_ids": [], "last_run": None}

seen = set(state.get('seen_ids', []))

# Also check existing research files for job IDs in URLs
research_dir = 'projects/job-search/research'
for filepath in glob.glob(f'{research_dir}/*.md'):
    with open(filepath) as f:
        content = f.read()
    for m in __import__('re').finditer(r'linkedin\.com/jobs/view/(\d+)', content):
        seen.add(m.group(1))

# Filter
with open('/tmp/linkedin_jobs_raw.json') as f:
    jobs = json.load(f)

new_jobs = [j for j in jobs if j['job_id'] not in seen]
already_seen = len(jobs) - len(new_jobs)

with open('/tmp/linkedin_jobs_new.json', 'w') as f:
    json.dump(new_jobs, f, indent=2)

print(f"Total: {len(jobs)} | Already seen: {already_seen} | New: {len(new_jobs)}")
EOF
```

Read `/tmp/linkedin_jobs_new.json` with the Read tool. If it's empty, skip to Step 7 (summary).

### Step 4 — Open Each Job Link and Read the Full Posting

For every job in `/tmp/linkedin_jobs_new.json`, use WebFetch to open its LinkedIn URL and extract the full job description. Fetch all jobs in parallel.

For each job page, extract:
- Full job title (may differ from email subject line)
- Company name
- Location and workplace type (Remote / Hybrid / On-site)
- Seniority level
- Employment type (Full-time / Contract / etc.)
- Compensation range (if listed)
- Complete job description and responsibilities
- Required qualifications
- Preferred/nice-to-have qualifications

**If a job page fails to load** (LinkedIn login wall, 404, redirect): fall back to the email stub data (title + company from email) and set `fetch_failed: true` on that job. Still score it based on available info.

WebFetch prompt to use for each job:
> "Extract: job title, company name, location, workplace type (remote/hybrid/on-site), seniority level, employment type, salary/compensation if listed, full job description including responsibilities and all qualifications (required and preferred). Return as structured text."

### Step 5 — Score Each Job

Scoring has two parts: (A) general fit signals, and (B) specific requirements check.

#### Part A — General Fit Signals

Score each job **High / Medium / Low** against James's profile (`context/me.md`) and job search criteria (`projects/job-search/README.md`). The README has the authoritative criteria — use it. Key signals summarized below for reference:

##### High — Strong fit, create task
- Title: Sr. PM, Principal PM, Staff PM, Director of Product, Head of Product
- Location: Remote OR hybrid in Northern VA / DC metro
- Industry: FinTech, payments, lending, BNPL, real estate (PropTech, brokerage tech, rental platforms, mortgage, title, home services — the full sector), marketplace, platform, B2B SaaS, clean tech, ag tech
- Product is customer-facing and/or mission-driven (provides genuine net benefit — not purely transactional or back-office)
- Compensation listed ≥ $150K (absence of comp info is neutral, not negative)
- Posting visibly highlights work-life balance, generous PTO, extra holidays, summer Fridays, or similar — companies that lead with this signal they mean it

##### Medium — Possible fit, create task
- Title unlisted or "Product Manager" without senior modifier, but company/scope suggests senior-level work
- Pay likely $130–150K but role has strong flexibility, mission appeal, or exciting product (aligns with James's long-term goal of eventually going full-time on Petit Puffin Play)
- Industry adjacent: health tech, ed tech, e-commerce, creator economy, climate tech, sustainability
- Hybrid outside Northern VA / DC but otherwise excellent fit — flag the commute in the task description

##### Low — Skip, mark seen
- Entry-level, associate PM, or IC technical PM requiring engineering background
- On-site anywhere, or hybrid outside Northern VA / DC metro
- Defense, intelligence, or federal government
- Purely internal / back-office product with no external users
- No meaningful industry overlap and pay likely below $130K

#### Part B — Specific Requirements Check

Job postings mix generic PM boilerplate with a handful of genuinely specific requirements. The generic stuff (e.g., "cross-functional collaboration", "data-driven mindset", "agile experience") is background noise — ignore it for scoring purposes.

**What counts as specific:** named industries, named product types, named metrics or outcomes, named markets or customer segments, or named methodologies that aren't universal. Examples:
- "experience with trial-to-paid conversion" — specific
- "managed high-growth SaaS products in enterprise productivity or collaboration markets" — specific
- "strong cross-functional communication" — generic, skip

For each posting, extract only the specific requirements and evaluate James's fit against each one using `context/me.md`:

| Specific Requirement | James's Fit | Notes |
|---|---|---|
| [requirement] | Strong / Partial / Gap | [brief rationale] |

**Let specific gaps pull the score down.** If Part A scores High but Part B reveals a meaningful gap on a specific requirement, the final score should be Medium (or Low if the gap is fundamental). A clean match on the specifics reinforces a High score. Surface this table in the Todoist task description so James can see the reasoning.

### Step 6 — Create Todoist Tasks

For **High** and **Medium** fit jobs only. Cap at 8 tasks per run.

Prioritize: High fit first, then Medium. Sort within each tier by how compelling the company/role is.

For each task:

```
Name: Research [Company] — [Role Title]
Description:
[Job URL]

Fit: [High / Medium]
[1–3 bullet points on general fit signals — industry, location, comp, title, mission]

Specific requirements:
• [Specific requirement] — [Strong match / Partial / Gap: one-line rationale]
• [Specific requirement] — [Strong match / Partial / Gap: one-line rationale]
(include all specific requirements found; omit generic boilerplate)

Project: Job Search ([YOUR_JOB_SEARCH_PROJECT_ID])
Section: Prep & Research ([YOUR_PREP_RESEARCH_SECTION_ID])
Priority: p2 for High, p3 for Medium
Due: today
```

If a posting has no specific requirements (pure boilerplate), note "No role-specific requirements found — verify before applying." in place of the specifics section.

Use the `mcp__todoist__todoist_task_create` tool. Create all tasks in parallel.

**Label**: If a `job-scan` label exists in Todoist, apply it. If not, skip — don't create it mid-run.

### Step 7 — Update State File

Append all newly seen job IDs (regardless of fit score) to `seen_ids`, and update `last_run`:

```python
python3 << 'EOF'
import json, os
from datetime import date

state_path = 'projects/job-search/job-scan-state.json'
if os.path.exists(state_path):
    with open(state_path) as f:
        state = json.load(f)
else:
    state = {"seen_ids": [], "last_run": None}

with open('/tmp/linkedin_jobs_new.json') as f:
    new_jobs = json.load(f)

new_ids = [j['job_id'] for j in new_jobs]
state['seen_ids'] = list(set(state.get('seen_ids', []) + new_ids))
state['last_run'] = str(date.today())

with open(state_path, 'w') as f:
    json.dump(state, f, indent=2)
print(f"State updated: {len(state['seen_ids'])} total seen IDs")
EOF
```

### Step 8 — Label Processed Emails in Gmail

Apply a `job scan` label to every LinkedIn alert email that was processed this run, so James can see at a glance which emails the EA has already reviewed when cleaning his inbox.

```python
python3 << 'EOF'
import subprocess, json

with open('/tmp/linkedin_message_ids.json') as f:
    message_ids = json.load(f)

def get_or_create_label(name):
    # List existing labels
    r = subprocess.run(
        ['gws', 'gmail', 'users', 'labels', 'list', '--params', '{"userId":"me"}'],
        capture_output=True, text=True
    )
    out = r.stdout
    start = out.find('{')
    if start >= 0:
        data = json.loads(out[start:])
        for label in data.get('labels', []):
            if label.get('name', '').lower() == name.lower():
                return label['id']
    # Create it if not found
    create_r = subprocess.run(
        ['gws', 'gmail', 'users', 'labels', 'create', '--params', '{"userId":"me"}',
         '--json', json.dumps({"name": name, "labelListVisibility": "labelShow", "messageListVisibility": "show"})],
        capture_output=True, text=True
    )
    out2 = create_r.stdout
    start2 = out2.find('{')
    if start2 >= 0:
        created = json.loads(out2[start2:])
        return created.get('id')
    return None

label_id = get_or_create_label('job scan')
if not label_id:
    print("Could not get or create 'job scan' label — skipping")
else:
    ok = 0
    for mid in message_ids:
        r = subprocess.run(
            ['gws', 'gmail', 'users', 'messages', 'modify',
             '--params', f'{{"userId":"me","id":"{mid}"}}',
             '--json', json.dumps({"addLabelIds": [label_id]})],
            capture_output=True, text=True
        )
        if r.returncode == 0:
            ok += 1
        else:
            print(f"  Failed to label {mid}")
    print(f"Labeled {ok}/{len(message_ids)} emails with 'job scan'")
EOF
```

Note: This labels the emails but does **not** mark them as read, archive, or modify them in any other way.

### Step 9 — Print Summary

```
Job Scan — [YYYY-MM-DD]
  Emails scanned:     X
  Listings found:     Y
  Already seen:       Z
  New listings:       N
  Tasks created:      M  (High: H, Medium: M)
  Skipped (low fit):  P
```

If no new listings were found: "No new listings since last scan."

---

## Rules

- Never send email. Never mark emails as read. Never archive.
- Create Todoist tasks directly — no approval step.
- Cap at 8 tasks per run. Better to surface fewer, higher-quality matches.
- Always update the state file, even if no tasks were created.
- If a job page is behind a login wall, fall back to email stub data — do not skip the job entirely.
- Do not create a research file (`.md`) — that's the `/research` skill's job. This skill only creates Todoist tasks.
