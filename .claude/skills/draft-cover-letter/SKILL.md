# Draft Cover Letter

Generate a tailored, polished cover letter for a specific job application — grounded in the existing research file and your career context.

---

## How to Invoke

```
/draft-cover-letter [company-name or research-file-path]
```

---

## Prerequisites

- A research file exists in `projects/job-search/research/` for the target role
- `context/me.md` — career narrative and background (always loaded)

---

## Workflow

### Step 1: Identify the Research File

- If invoked with a clear company name or file path, locate the matching file in `projects/job-search/research/`
- If ambiguous or not provided, list all available research files and ask to choose
- If no research file exists, prompt to run `/research <job-posting-URL>` first

### Step 2: Load Context (silent)

Read:
- The research file (especially: Company Overview, Role Analysis, Key Talking Points, Fit tables)
- `context/me.md` (career themes, accomplishments, skills)

### Step 3: Ask Customization Questions

Before drafting, ask **3 focused questions**:

1. **Angle** — Is there a specific aspect of your background you want to lead with? Or should I choose based on the research?
2. **Omit / downplay** — Anything you want to leave out or not emphasize?
3. **Tone** — Conversational and direct, or more formal? (Default: confident and direct, not stiff)

### Step 4: Draft the Cover Letter

**Header (always include):**
```
[Your Name]
[Phone] | [Email]
[Current date]
```

**Salutation:**
```
Dear [Company] Hiring Team,
```

**Closing (always use):**
```
Sincerely,
[Your first name]
```

| Section | Content |
|---------|---------|
| **Opening** | A specific hook — reference the company's mission, a recent development, or a genuine insight. NOT "I am writing to apply for..." |
| **Body 1 — Why this company** | 2-3 sentences: what drew you to this specific company/role. Use a concrete insight from the research. |
| **Body 2 — Why you** | 2-3 talking points grounded in specific accomplishments with numbers. Mirror language from the job posting. |
| **Body 3 — Forward value** | 1-2 sentences: what you'll bring. Forward-looking, confident, specific. |
| **Closing** | Genuine enthusiasm + clear CTA. Not desperate. |

**Format rules:**
- 300–400 words target
- Specific accomplishments with metrics where possible
- Mirror key language from the job posting
- No generic filler phrases

### Step 5: Proofread (silent)

Review for spelling, grammar, awkward phrasing, and repetition. Fix silently.

### Step 6: Length Check (silent)

- Target: 300–400 words
- If over 400: tighten the weakest sentences
- If under 300: check if substance is missing

### Step 7: Present Draft

Show the full letter, then below it:

```
---
**Word count:** XXX words
**Proofreading:** [corrections made, or "No issues found"]
**Length:** [cuts made, or "Within target range"]
```

Ask: *"Any feedback or changes? Reply 'done' when you're happy with it."*

### Step 8: Iterate

Incorporate feedback and re-present the full revised letter. Repeat until confirmed done.

### Step 9: Save Output

Write the final letter to:
```
projects/job-search/cover-letters/YYYY-MM-DD-[company-slug]-cover-letter.md
```

### Step 10: Export to Google Drive

Use `gws` CLI via Bash to create the Google Doc in Drive.

**10a — Identify your job search folder in Drive:**
```bash
gws drive files list --params '{"q": "name = \"[Your Job Search Folder]\" and mimeType = \"application/vnd.google-apps.folder\"", "fields": "files(id,name)"}'
```

**10b — Check for existing company folder:**
```bash
gws drive files list --params '{"q": "name = \"[Company]\" and mimeType = \"application/vnd.google-apps.folder\" and \"PARENT_FOLDER_ID\" in parents", "fields": "files(id,name)"}'
```

**10c — Create company folder if it does not exist:**
```bash
gws drive files create --json '{"name": "[Company]", "mimeType": "application/vnd.google-apps.folder", "parents": ["PARENT_FOLDER_ID"]}'
```

**10d — Create the Google Doc:**
```bash
gws drive files create --json '{"name": "[Company] — [Role Title] Cover Letter", "mimeType": "application/vnd.google-apps.document", "parents": ["FOLDER_ID"]}'
```

**10e — Insert cover letter text:**
```bash
gws docs documents batchUpdate --params '{"documentId": "DOC_ID"}' --json '{"requests": [{"insertText": {"location": {"index": 1}, "text": "[full letter text with \\n for newlines]"}}]}'
```

**10f — Apply formatting** (Spectral font, 11pt, line spacing 100). Get end index from a `gws docs documents get` call first, use `(endIndex - 1)`:
```bash
gws docs documents batchUpdate --params '{"documentId": "DOC_ID"}' --json '{
  "requests": [
    {"updateTextStyle": {"range": {"startIndex": 1, "endIndex": END_INDEX}, "textStyle": {"weightedFontFamily": {"fontFamily": "Spectral", "weight": 400}, "fontSize": {"magnitude": 11, "unit": "PT"}}, "fields": "weightedFontFamily,fontSize"}},
    {"updateParagraphStyle": {"range": {"startIndex": 1, "endIndex": END_INDEX}, "paragraphStyle": {"lineSpacing": 100}, "fields": "lineSpacing"}}
  ]
}'
```

Confirm the Drive doc link on completion.

---

## Output Format

```markdown
# Cover Letter — [Company] | [Role Title]

**Date:** YYYY-MM-DD
**Role:** [Title]
**Company:** [Company]
**Research file:** [path]
**Status:** Final

---

[Full cover letter text]
```

---

## Rules

- Never start with "I am writing to apply for..." or any variant
- Always ground talking points in specific accomplishments — no vague claims
- Mirror language from the job posting
- Keep it to one page (300–400 words)
- If the research file flags concerns (comp mismatch, culture red flags), do NOT surface them in the letter
- If no research file exists, recommend running `/research` first
