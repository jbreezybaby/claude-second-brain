# Sync Public Repo

Sync shareable changes from your private EA repo to a public GitHub repo (template/portfolio).

---

## How to Invoke

```
/sync-public-repo
```

---

## Repo Locations

| Repo | Path |
|------|------|
| Private (this EA) | `/path/to/your/EA` |
| Public (template) | `/path/to/your/public-repo` |
| GitHub | `https://github.com/[your-username]/[your-repo]` |

Update these paths in your local copy of this skill.

---

## What Gets Synced

| Sync | Don't Sync |
|------|-----------|
| `.claude/skills/*/SKILL.md` — all skills | `context/` — personal info |
| `.claude/rules/` | `projects/` — personal workstreams |
| `CLAUDE.md` — but templatized | `decisions/log.md` — personal decisions |
| `references/sops/` | `archives/` — personal |
| `templates/` | `.mcp.json`, `.env`, `CLAUDE.local.md` |
| `README.md` if updated | Any cover letters or research files |

---

## Workflow

### Step 1: Identify what changed

Review recent changes in the private repo:

```bash
git log --oneline -10
git diff HEAD~5 --name-only
git status --short
```

Determine which files are sync-worthy (skills, sops, rules, templates).

### Step 2: For each changed skill — copy and strip personal info

Copy the skill file to the public repo, then review and strip any personal details:
- Replace specific names with `[Your Name]` / `[Partner Name]`
- Replace specific Todoist project/section IDs with a note to fetch them dynamically
- Replace specific Google Drive folder IDs with `[YOUR_FOLDER_ID]`
- Replace specific email addresses, phone numbers, personal URLs
- Replace specific company/project names with `[Your Company]`
- Replace property addresses or personal details with generic placeholders

### Step 3: For CLAUDE.md — templatize before copying

CLAUDE.md contains personal tool integrations and project names. Replace:
- GCP project IDs → `[YOUR_GCP_PROJECT]`
- Personal project names → generic equivalents
- Personal tool details → template placeholders

### Step 4: Commit and push

```bash
cd /path/to/your/public-repo
git add .
git status  # review before committing
git commit -m "Sync: [brief description of what changed]"
git push origin main
```

### Step 5: Confirm

Report what was synced and the GitHub URL.

---

## Rules

- **Always review for personal info before committing** — names, IDs, addresses, emails, phone numbers
- Never copy personal context files (`context/`, `projects/`, `decisions/`, `archives/`)
- Never copy `.mcp.json`, `.env`, or `CLAUDE.local.md`
- If a new skill folder doesn't exist in the public repo yet, create it first
- Keep the public repo's `README.md` current if new skills are added — update the skills table
- Always show a summary of what will be synced and get approval before pushing
