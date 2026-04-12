# Log Job Application

Log a completed job application to a Job Contact Log Google Sheet. Automatically fills required fields from research files and web search, prompts only for what it can't find, and appends a row to the sheet for use in weekly unemployment certifications.

---

## How to Invoke

```
/log-job-application [company-name | research-file-slug | job-posting-URL]
```

**Examples:**
```
/log-job-application acme-corp
/log-job-application projects/job-search/research/2026-04-02-acme-corp-pm.md
/log-job-application (no arg — will prompt)
```

---

## Prerequisites

- `gws` CLI authenticated (`gws auth status` should show your email)
- Job Contact Log sheet exists OR this is the first run (skill creates it automatically)

---

## Workflow

### Step 1 — Identify the Application

- If arg matches a path or slug in `projects/job-search/research/` → load that research file
- If arg is a company name → fuzzy-search `projects/job-search/research/` for a match (by filename)
- If arg is a URL → treat as job posting URL; no research file
- If no arg → ask: company name and role title before proceeding

### Step 2 — Extract Known Data from Research File (if exists)

Read the research file silently. Extract:

| Field | Where to Find |
|-------|---------------|
| Company name | Company Overview table |
| HQ address | Company Overview — HQ row |
| Role title | Role Analysis table |
| Job posting URL | Header |
| Date applied | Today's date |

### Step 3 — Web Search for Missing Required Fields

For any required field not found in the research file, search the web. Use tools in this priority order:
1. `perplexity_ask` (if Perplexity MCP connected)
2. `WebSearch` + `WebFetch`

| Missing Field | Search Strategy |
|---------------|----------------|
| Company address | Search "[company name] headquarters address" |
| HR contact email | Search "[company name] careers email" or fetch their careers page |
| Contact person | Default to "Talent Acquisition Team" if nothing findable — never fabricate a name |

Result field always defaults to: **"Application submitted online"**

### Step 4 — Present Prefilled Table and Prompt for Gaps

Show what was auto-filled vs. what's missing or uncertain:

```
| Field              | Value                        | Source          |
|--------------------|------------------------------|-----------------|
| Date Applied       | YYYY-MM-DD                   | Auto            |
| Company            | [name]                       | Research file   |
| Address            | [address or ⚠️ MISSING]      | Web search      |
| Contact Person     | Talent Acquisition Team      | Default         |
| Phone / Email      | [email or ⚠️ MISSING]        | Web search      |
| Position           | [role title]                 | Research file   |
| Result             | Application submitted online | Default         |
```

Ask to confirm, correct, or fill any gaps. Do not proceed until all 7 fields are confirmed.

### Step 5 — Confirm Full Row Before Writing

Show the final row as it will appear in the sheet and wait for explicit approval before writing.

### Step 6 — Write to Google Sheet

**Sheet name:** Job Contact Log
**Drive folder:** [YOUR_DRIVE_FOLDER_ID] — update in config
**Sheet ID storage:** `.claude/skills/log-job-application/config.md`

#### First-run setup (if no Sheet ID in config.md):

**6a — Create the sheet:**
```bash
gws drive files create --json '{"name": "Job Contact Log", "mimeType": "application/vnd.google-apps.spreadsheet", "parents": ["[YOUR_DRIVE_FOLDER_ID]"]}' 2>/dev/null
```

**6b — Write header row:**
```bash
gws sheets spreadsheets values update \
  --params '{"spreadsheetId": "SHEET_ID", "range": "Sheet1!A1:K1", "valueInputOption": "USER_ENTERED"}' \
  --json '{"values": [["Date Applied", "Company", "Full Address", "Contact Person / Title", "Phone or Email", "Position Applied For", "Result", "Job Posting URL", "Research File", "Status", "Notes"]]}' \
  2>/dev/null
```

**6c — Save Sheet ID to config:**
Write spreadsheet ID to `.claude/skills/log-job-application/config.md` for reuse on future runs.

#### Append row (all runs):

```bash
gws sheets spreadsheets values append \
  --params '{"spreadsheetId": "SHEET_ID", "range": "Sheet1!A:K", "valueInputOption": "USER_ENTERED", "insertDataOption": "INSERT_ROWS"}' \
  --json '{"values": [["DATE", "COMPANY", "ADDRESS", "CONTACT", "PHONE_OR_EMAIL", "POSITION", "RESULT", "JOB_URL", "RESEARCH_FILE", "Applied", ""]]}' \
  2>/dev/null
```

**Sheet columns:**

| Col | Field | Default / Notes |
|-----|-------|-----------------|
| A | Date Applied | Today's date |
| B | Company | Full legal name |
| C | Full Address | Street, city, state, zip |
| D | Contact Person / Title | "Talent Acquisition Team" for online apps |
| E | Phone or Email | Careers page URL or HR email |
| F | Position Applied For | Exact job title |
| G | Result | "Application submitted online" |
| H | Job Posting URL | From research file or input |
| I | Research File | Relative path, or blank |
| J | Status | "Applied" on creation |
| K | Notes | Blank on creation |

### Step 7 — Output

Confirm completion with:
- Link to the Google Sheet
- Count of applications logged this week (Sunday–Saturday) — flag if below 2 for the week (minimum required for most state unemployment certifications)

---

## Rules

- Never fabricate a contact name — use "Talent Acquisition Team" as the default
- Never write to the sheet without explicit confirmation in Step 5
- Always report this week's application count
- If gws Sheets commands fail, output the full row as a formatted table so it can be pasted manually — do not silently skip logging
- Always use `2>/dev/null` on gws calls to suppress keyring header

---

## References

- Research files: `projects/job-search/research/`
- Sheet config: `.claude/skills/log-job-application/config.md`
