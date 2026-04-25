# Push Private Repo

Stage, commit, and push all local changes in the EA repo to the private GitHub repo.

---

## How to Invoke

```
/push-private-repo [optional commit message]
```

---

## Repo

| | |
|---|---|
| **Local path** | `[YOUR_EA_LOCAL_PATH]` |
| **GitHub** | `[YOUR_PRIVATE_GITHUB_REPO_URL]` |
| **Branch** | `main` |

---

## Workflow

### Step 1: Check status

```bash
git status
git diff --stat
```

If there are no changes (clean working tree and nothing staged), report that and stop — no commit needed.

### Step 2: Draft a commit message

Review what changed and write a concise commit message:
- One short subject line (under 60 chars)
- Focus on *what changed* — e.g., "Add push-private-repo skill", "Update job search context", "Log Tilt application"
- If the user provided a message when invoking, use that verbatim

### Step 3: Stage, commit, and push

```bash
git add -A
git commit -m "MESSAGE"
git push origin main
```

### Step 4: Confirm

Report the commit SHA and confirm the push succeeded. Include the GitHub URL:
`[YOUR_PRIVATE_GITHUB_REPO_URL]`

---

## Rules

- Never commit `.env`, `CLAUDE.local.md`, `.claude/settings.local.json`, or `.mcp.json` — these are in `.gitignore` and should stay out
- If there are no changes, stop and report — do not create an empty commit
- If the user passes a message at invocation (e.g., `/push-private-repo Add new skill`), use it verbatim
- Keep commit messages factual — describe what changed, not why
