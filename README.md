# Claude Second Brain

A personal executive assistant and second brain built on [Claude Code](https://claude.ai/code). Combines **GTD** (Getting Things Done) for task management with **PARA** (Projects, Areas, Resources, Archives) for information architecture.

---

## What This Is

This repo is the working directory for a Claude Code session that acts as a persistent EA. It connects to your tools, knows your context, and runs structured workflows (called *skills*) to handle recurring tasks — inbox triage, morning briefings, research, cover letters, and more.

The system gets smarter over time through:
- **Context files** (`context/`) — who you are, what you're working on, your goals
- **Skills** (`.claude/skills/`) — reusable workflows invoked with slash commands
- **Memory** — Claude Code's persistent memory layer (stored in `~/.claude/projects/`)
- **Decision log** (`decisions/log.md`) — append-only log of meaningful choices

---

## Structure

```
claude-second-brain/
├── CLAUDE.md                    # Core instructions for the EA
├── context/                     # Who you are and what you're working on
│   ├── me.md                    # Your background, experience, priorities
│   ├── work.md                  # Work and business context
│   ├── team.md                  # People you work with
│   ├── areas.md                 # Ongoing areas of responsibility
│   ├── current-priorities.md    # What matters right now
│   └── goals.md                 # Quarterly goals
├── projects/                    # Active workstreams (one folder each)
├── archives/                    # Completed or inactive material
├── references/
│   ├── sops/                    # System design and operating procedures
│   └── examples/                # Style guides and example outputs
├── decisions/
│   └── log.md                   # Append-only decision log
├── templates/
│   └── session-summary.md       # End-of-session recap template
└── .claude/
    ├── rules/                   # Always-on behavior rules
    └── skills/                  # Slash command workflows
        ├── research/
        ├── draft-cover-letter/
        ├── inbox-triage-todoist/
        ├── inbox-triage-gmail/
        ├── inbox-triage-calendar/
        ├── morning-briefing/
        └── weekly-review/
```

---

## Skills

Skills are reusable workflows stored in `.claude/skills/`. Invoke them with `/skill-name` in a Claude Code session.

| Skill | Command | Description |
|-------|---------|-------------|
| Research | `/research <URL or topic>` | Job posting or general topic research |
| Draft Cover Letter | `/draft-cover-letter <company>` | Tailored cover letter from a research file |
| Inbox Triage — Todoist | `/inbox-triage-todoist` | GTD Clarify + Organize for Todoist inbox |
| Inbox Triage — Gmail | `/inbox-triage-gmail` | Scan Gmail (read-only) and surface tasks |
| Inbox Triage — Calendar | `/inbox-triage-calendar` | Scan calendar and create prep tasks |
| Morning Briefing | `/morning-briefing` | Daily orientation — calendar, tasks, focus |
| Weekly Review | `/weekly-review` | Full GTD weekly review (10-step) |

---

## Tool Integrations

| Tool | Purpose | How |
|------|---------|-----|
| Todoist | Task management (single source of truth) | MCP server (`@greirson/mcp-todoist`) |
| Gmail | Read-only inbox scanning | `gws` CLI (Bash tool) |
| Google Calendar | Event and prep task creation | `gws` CLI (Bash tool) |
| Google Drive / Docs / Sheets | File creation and management | `gws` CLI (Bash tool) |

### Google Workspace (`gws` CLI)

This system uses the [`@googleworkspace/cli`](https://github.com/googleworkspace/cli) tool instead of an MCP server for Google Workspace access. Claude calls `gws` commands via the Bash tool.

Install: `brew install googleworkspace-cli`
Auth: `gws auth setup && gws auth login`

---

## Task Management Design

**GTD + PARA.** Full design: `references/sops/gtd-para-system.md`

| GTD Step | Tool | Skill |
|----------|------|-------|
| Capture | Todoist Inbox + Gmail + Calendar | — |
| Clarify + Organize | Todoist projects/sections | `/inbox-triage-*` |
| Reflect | Weekly + daily review | `/weekly-review`, `/morning-briefing` |
| Engage | Prioritized task list | — |

**Gmail is read-only by design.** The EA drafts responses but never sends, replies, or deletes.

---

## Getting Started

1. Clone this repo as your working directory
2. Fill in `context/me.md`, `context/work.md`, and `context/current-priorities.md`
3. Set up Todoist MCP server in `.mcp.json`
4. Install and authenticate `gws` CLI
5. Open Claude Code from this directory
6. Run `/morning-briefing` to start

---

## Philosophy

- **One source of truth** — Todoist owns all tasks. Nothing lives in two places.
- **Skills over one-offs** — When a workflow repeats, build a skill.
- **Context over prompts** — The more context Claude has, the less you have to explain each session.
- **Archives, don't delete** — Move completed material to `archives/`. Deleting loses history.
