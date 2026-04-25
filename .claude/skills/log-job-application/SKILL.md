# Log Job Application

Log a completed job application to the VEC Job Contact Log Google Sheet. Automatically fills required fields from research files and web search, prompts only for what it can't find, and appends a row to the sheet for use in weekly VEC certifications.

---

## How to Invoke

```
/log-job-application [company-name | research-file-slug | job-posting-URL]
```

**Examples:**
```
/log-job-application energyhub
/log-job-application projects/job-search/research/2026-04-02-energyhub-lead-pm.md
/log-job-application (no arg — will prompt)
```

---

## Prerequisites

- `gws` CLI authenticated (`gws auth status` should show `[YOUR_EMAIL]`)
- VEC Job Contact Log sheet exists OR this is the first run (skill creates it automatically)

---

## Workflow

### Step 1 — Identify the Application

- If arg matches a path or slug in `projects/job-search/research/` → load that research file
- If arg is a company name → fuzzy-search `projects/job-search/research/` for a match (by filename)
- If arg is a URL → treat as job posting URL; no research file
- If no arg → ask James: company name and role title before proceeding

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

For any VEC-required field not found in the research file, search the web. Use tools in this priority order:
1. `perplexity_ask` (if Perplexity MCP connected)
2. `WebSearch` + `WebFetch`

| Missing Field | Search Strategy |
|---------------|----------------|
| Company address | Search "[company name] headquarters address" |
| Company phone | Search "[company name] headquarters phone number" or fetch their contact page — **required by VEC; must be a real phone number, not a URL** |
| HR contact email | Search "[company name] careers email" or fetch their careers page |
| Contact person | Default to "Talent Acquisition Team" if nothing findable — never fabricate a name |

Result field always defaults to: **"Application submitted online"**

### Step 4 — Compile Row and Write to Google Sheet

Assemble the row using all fields gathered in Steps 2–3:

| Col | Field | Value Source |
|-----|-------|-------------|
| A | Date Applied | Today's date (YYYY-MM-DD) |
| B | Company | From research file or input |
| C | Full Address | From research file HQ row, or web search result |
| D | Contact Person / Title | "Talent Acquisition Team" (default) or found contact |
| E | Phone or Email | Phone number preferred (required by VEC); use HR email or careers URL if no phone found |
| F | Position Applied For | Role title from research file or input |
| G | Result | "Application submitted online" |
| H | Job Posting URL | From research file header or input |
| I | Research File | Relative path (e.g. `projects/job-search/research/YYYY-MM-DD-company-role.md`), or blank |
| J | Status | "Applied" |
| K | Notes | Blank |

If neither a phone number nor email/URL can be found after web search, stop and ask James before proceeding.

Write immediately — no confirmation needed:

### Step 5 — Write to Google Sheet

**Sheet name:** VEC Job Contact Log
**Drive folder:** Unemployment Claims (ID: `[YOUR_UNEMPLOYMENT_CLAIMS_FOLDER_ID]`)
**Sheet ID storage:** `.claude/skills/log-job-application/config.md`

#### First-run setup (if no Sheet ID in config.md):

**6a — Create the sheet:**
```bash
gws drive files create --json '{"name": "VEC Job Contact Log", "mimeType": "application/vnd.google-apps.spreadsheet", "parents": ["[YOUR_UNEMPLOYMENT_CLAIMS_FOLDER_ID]"]}' 2>/dev/null
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
| E | Phone | Company headquarters phone — **required by VEC** |
| F | Email / URL | HR email or careers page URL |
| G | Position Applied For | Exact job title |
| H | Result | "Application submitted online" |
| I | Job Posting URL | From research file or input |
| J | Research File | Relative path, or blank |
| K | Status | "Applied" on creation |
| L | Notes | Blank on creation |

### Step 7 — Output

Confirm completion with:
- Link to the Google Sheet
- Count of applications logged this week (Sunday–Saturday) — flag if below 2 for the week

---

## Rules

- Never fabricate a contact name — use "Talent Acquisition Team" as the default
- Always report this week's application count (VEC requires 2 minimum per week)
- If gws Sheets commands fail, output the full row as a formatted table so James can paste it manually — do not silently skip logging
- Always use `2>/dev/null` on gws calls to suppress keyring header

---

## References

- Research files: `projects/job-search/research/`
- Sheet config: `.claude/skills/log-job-application/config.md`
- Drive folder (Unemployment Claims): `[YOUR_UNEMPLOYMENT_CLAIMS_FOLDER_ID]`
- VEC requirements: `projects/unemployment/README.md`
