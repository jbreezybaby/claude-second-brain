# Research Skill

Conduct structured research and produce a written report. Supports two modes:

- **Job Posting Research** — given a job posting URL or pasted text, research the company, role, and interview landscape
- **General Research** — given a topic, produce a structured briefing

---

## How to Invoke

```
/research <job-posting-URL>
/research <pasted job posting text>
/research <topic description>
```

If the input is a URL or contains job posting content (title, responsibilities, qualifications), treat it as **Job Posting Research**. Otherwise, treat it as **General Research**.

---

## Tools

Use the best available search tools, in this priority order:

| Priority | Tool | When Available |
|----------|------|----------------|
| 1 | `perplexity_ask` (Sonar Pro) | When Perplexity MCP is connected |
| 2 | `perplexity_reason` (Sonar Reasoning Pro) | For complex/multi-step questions when Perplexity MCP is connected |
| 3 | `WebSearch` + `WebFetch` | Always available (built-in) |

When using WebSearch/WebFetch, compensate by running multiple targeted queries and cross-referencing results.

---

## Research Methodology

### Step 1: Understand the Input
- If URL: fetch the page with WebFetch and extract all relevant details
  - **If the fetch fails** (403, redirect, timeout, or empty content) and this looks like a job posting URL: immediately stop and ask the user to paste the job posting text before proceeding. Do not attempt to research the role without the posting content.
- If pasted text: parse it directly
- If topic: clarify scope if ambiguous

### Step 2: Research (run searches in parallel where possible)
- Primary information (company overview, topic fundamentals)
- Secondary context (recent news, trends, competitors)
- Practical/actionable information (specific to the mode — see templates below)

### Step 3: Synthesize
- Cross-reference findings across sources
- Flag anything outdated or contradictory
- Identify gaps — call them out honestly rather than guessing

### Step 4: Write the Report
- Use the appropriate template below
- Save to the correct location
- Include a Sources section with all URLs used

---

## Job Posting Research

### Save Location
`projects/job-search/research/YYYY-MM-DD-company-role-slug.md`

### Report Template

```markdown
# [Company Name] — [Role Title]

**Date:** YYYY-MM-DD
**Posting URL:** [link]
**Status:** Research Complete

---

## Company Overview

| | |
|---|---|
| **Company** | Name |
| **Industry** | ... |
| **Size** | Employee count, stage (startup/growth/enterprise) |
| **Founded** | Year |
| **HQ** | Location |
| **Funding / Revenue** | Latest known info |
| **Key People** | CEO, hiring manager if findable |

### What They Do
2-3 sentence plain-English summary. What problem do they solve? For whom?

### Recent News
| Date | Headline | Source |
|------|----------|--------|
| ... | ... | [link] |

### Culture & Reputation
Summary of Glassdoor reviews, public sentiment, notable culture traits. Be honest — flag red flags if they exist.

---

## Role Analysis

| | |
|---|---|
| **Title** | ... |
| **Level** | Junior / Mid / Senior / Lead / Director |
| **Team** | If mentioned |
| **Location** | Remote / Hybrid / On-site, and where |
| **Compensation** | If listed or findable |

### Key Responsibilities
Bulleted list extracted from the posting.

### Required Qualifications
| Requirement | Your Fit | Notes |
|-------------|----------|-------|
| ... | Strong / Partial / Gap | ... |

(Reference @context/me.md to assess fit.)

### Nice-to-Haves
| Item | Your Fit | Notes |
|------|----------|-------|
| ... | ... | ... |

---

## Interview Prep

### Likely Interview Topics
| Topic | Why It's Likely | Prep Notes |
|-------|-----------------|------------|
| ... | ... | ... |

### Questions to Ask
5-7 thoughtful questions tailored to this specific role and company. Not generic.

### Key Talking Points
3-5 specific things to emphasize from your background that map to this role.

---

## Strategic Notes

- Overall fit assessment (1-2 sentences, be direct)
- Any concerns or things to investigate further
- Suggested next steps

---

## Sources
- [Source Title](URL)
```

---

## General Research

### Save Location
`research/YYYY-MM-DD-topic-slug.md`

### Report Template

```markdown
# Research: [Topic]

**Date:** YYYY-MM-DD
**Query:** What was asked

---

## Summary
3-5 sentence executive summary. What did you find? What's the bottom line?

---

## Key Findings

### [Finding 1]
Details, context, evidence.

### [Finding 2]
Details, context, evidence.

### [Finding 3]
Details, context, evidence.

(Use as many sections as needed. Use tables when comparing options or listing structured data.)

---

## Recommendations
What to do with this information. Be specific and actionable.

---

## Open Questions
Things that couldn't be answered or need further investigation.

---

## Sources
- [Source Title](URL)
```

---

## Rules

- Always include the Sources section. Every claim should be traceable.
- Use tables for structured data.
- Be direct and efficient. Don't pad the report with filler.
- If information is uncertain or couldn't be verified, say so explicitly.
- For job research: assess fit against actual background (@context/me.md), not a hypothetical candidate.
- Create the output directory if it doesn't exist before saving.
- **Never ask for permission to access publicly available web pages.** Fetch URLs, run searches, and follow redirects autonomously. Only pause if a page requires authentication or a fetch hard-fails with no fallback.
