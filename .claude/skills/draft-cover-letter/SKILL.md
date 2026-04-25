# Draft Cover Letter

Generate a tailored, polished cover letter for a specific job application — grounded in the existing research file and James's career context.

---

## How to Invoke

```
/draft-cover-letter [company-name or research-file-path]
```

**Examples:**
```
/draft-cover-letter energyhub
/draft-cover-letter projects/job-search/research/2026-04-02-energyhub-lead-pm.md
```

---

## Prerequisites

- A research file exists in `projects/job-search/research/` for the target role
- `context/me.md` — career narrative and background (always loaded)

---

## Workflow

### Step 1: Identify the Research File

- If invoked with a clear company name or file path, locate the matching file in `projects/job-search/research/`
- If ambiguous or not provided, list all available research files and ask James to choose
- If no research file exists for the target role, prompt James to run `/research <job-posting-URL>` first

### Step 2: Load Context (silent)

Read the following — no user action needed:
- The research file (especially: Company Overview, Role Analysis, Key Talking Points, James's Fit tables)
- `context/me.md` (career themes, accomplishments, skills)

### Step 3: Ask About Angle

Before drafting, ask James one question:

**Angle** — Is there a specific aspect of your background you want to lead with or emphasize? (e.g., "0→1 builder", "fintech/regulatory depth", "real estate sector") Or should I choose based on the research?

Incorporate the answer into the draft before writing.

### Step 4: Draft the Cover Letter

Use the research file's **Key Talking Points** and **James's Fit** tables as the backbone. Structure:

**Header (always include):**
```
James Calabrese
[YOUR_PHONE] | [YOUR_EMAIL]
[Current date — e.g., April 2, 2026]
```

**Closing (always use):**
```
Sincerely,
James
```

| Section | Content |
|---------|---------|
| **Opening** | A specific hook — reference something about the company, their mission, or a recent development (from research). NOT "I am writing to apply for..." |
| **Body 1 — Why this company** | 2-3 sentences: what drew James to this specific company/role. Use a concrete insight from the research (recent news, mission angle, product specifics). |
| **Body 2 — Why James** | 2-3 talking points grounded in specific accomplishments with numbers where available. Mirror language from the job posting. Pull from Key Talking Points in research. |
| **Body 3 — Forward value** | 1-2 sentences: what James will bring in the first 90 days. Forward-looking, confident, specific. |
| **Closing** | Genuine enthusiasm + clear CTA. Not desperate. Example: "I'd welcome the opportunity to talk about how I can contribute." |

**Format rules:**
- 300–400 words target (one page as PDF at standard margins)
- Specific accomplishments with metrics where possible (pull from me.md)
- Mirror key language from the job posting
- No generic filler phrases

### Step 5: Proofread (silent)

Review the draft for:
- Spelling and grammatical errors → correct silently
- Awkward phrasing → smooth out
- Repetition → cut

### Step 6: Length Check (silent)

- Count words. Target: 300–400 words.
- If over 400: tighten the weakest sentences. Note what was cut and why.
- If under 300: check if substance is missing, not just word count.

### Step 7: Present Draft

Show James the full letter, then below it:

```
---
**Word count:** XXX words
**Proofreading:** [any corrections made, or "No issues found"]
**Length:** [any cuts made, or "Within target range"]
```

Then ask: *"Any feedback or changes? Reply 'done' when you're happy with it."*

### Step 8: Iterate

- Incorporate feedback and re-present the full revised letter
- Repeat until James confirms it's done (replies "done", "looks good", "send it", etc.)

### Step 9: Save Output

Write the final letter to:
```
projects/job-search/cover-letters/YYYY-MM-DD-[company-slug]-cover-letter.md
```

### Step 10: Export to Google Drive

Use `gws` CLI via Bash to create the Google Doc in Drive. The parent folder is the "[YOUR_JOB_HUNT_FOLDER_NAME]" folder (ID: `[YOUR_JOB_HUNT_FOLDER_ID]`).

**10a — Check for existing company folder:**
```bash
gws drive files list --params '{"q": "name = \"[Company]\" and mimeType = \"application/vnd.google-apps.folder\" and \"[YOUR_JOB_HUNT_FOLDER_ID]\" in parents", "fields": "files(id,name)"}'
```

**10b — Create company folder if it doesn't exist:**
```bash
gws drive files create --json '{"name": "[Company]", "mimeType": "application/vnd.google-apps.folder", "parents": ["[YOUR_JOB_HUNT_FOLDER_ID]"]}'
```

**10c — Create the Google Doc:**
```bash
gws drive files create --json '{"name": "[Company] — [Role Title] Cover Letter", "mimeType": "application/vnd.google-apps.document", "parents": ["[FOLDER_ID]"]}'
```

**10d — Insert cover letter text (include salutation inline):**

Include "Dear [Company] Hiring Team," directly in the text body — after the date line and before the first body paragraph. Do NOT insert the salutation as a separate step. Full text structure:

```
James Calabrese\n[YOUR_PHONE] | [YOUR_EMAIL]\n[Date]\n\nDear [Company] Hiring Team,\n\n[Body paragraphs separated by \n\n]\n\nSincerely,\nJames
```

```bash
gws docs documents batchUpdate --params '{"documentId": "[DOC_ID]"}' --json '{"requests": [{"insertText": {"location": {"index": 1}, "text": "[full letter text including salutation, with \\n for newlines]"}}]}'
```

**10e — Apply formatting (Spectral font, 11pt, line spacing 100):**

After Step 10d, get the document's current end index:
```bash
gws docs documents get --params '{"documentId": "[DOC_ID]"}' | python3 -c "import json,sys; doc=json.load(sys.stdin); content=doc.get('body',{}).get('content',[]); last=content[-1] if content else {}; print('endIndex:', last.get('endIndex', 'unknown'))"
```

Then apply formatting using `(endIndex - 1)` as the range end:
```bash
gws docs documents batchUpdate --params '{"documentId": "[DOC_ID]"}' --json '{
  "requests": [
    {"updateTextStyle": {"range": {"startIndex": 1, "endIndex": [END_INDEX]}, "textStyle": {"weightedFontFamily": {"fontFamily": "Spectral", "weight": 400}, "fontSize": {"magnitude": 11, "unit": "PT"}}, "fields": "weightedFontFamily,fontSize"}},
    {"updateParagraphStyle": {"range": {"startIndex": 1, "endIndex": [END_INDEX]}, "paragraphStyle": {"lineSpacing": 100}, "fields": "lineSpacing"}}
  ]
}'
```

**Notes:**
- Never insert the salutation as a separate batchUpdate request — doing so in the same call as formatting causes the endIndex to shift mid-batch, leaving the salutation text unformatted
- Always get a fresh endIndex from `gws docs documents get` after all text is inserted, before applying formatting

Confirm the Drive doc link to James on completion.

---

## Output Format

The saved markdown file should look like:

```markdown
# Cover Letter — [Company] | [Role Title]

**Date:** YYYY-MM-DD
**Role:** [Title]
**Company:** [Company]
**Research file:** [path]
**Status:** Draft / Final

---

[Full cover letter text]
```

---

## Rules

- Never start with "I am writing to apply for..." or any variant
- Always ground talking points in specific accomplishments — no vague claims
- Use language that mirrors the job posting
- Keep it to one page (300–400 words)
- If the research file flags concerns (e.g., comp mismatch, culture red flags), do NOT surface them in the cover letter — those are for James's decision-making only
- If James hasn't run `/research` for this role yet, say so and recommend doing that first

---

## References

- Career context: `context/me.md`
- Research files: `projects/job-search/research/`
- Output: `projects/job-search/cover-letters/`
- Google Drive parent folder: `[YOUR_JOB_HUNT_FOLDER_ID]` ([YOUR_JOB_HUNT_FOLDER_NAME])
