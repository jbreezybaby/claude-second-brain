# Schedule Bread

Build a bread-baking schedule and write it to Google Calendar as time-blocked events.

---

## How to Invoke

```
/schedule-bread
```

---

## Prerequisites

- `gws` CLI authenticated
- EA Schedule calendar ID hardcoded below

---

## Calendar IDs

| Calendar | ID |
|----------|----|
| EA Schedule | `[YOUR_EA_SCHEDULE_CALENDAR_ID]` |

---

## Workflow

### Step 1: Select Bread Type

Present the following options and wait for a response:

> **What are you baking?**
> 1. 40% Whole Wheat Straight Dough

Only one bread type exists currently. When the user selects, proceed to Step 2.

---

### Step 2: Batch Size

Ask:

> **How much dough are you making?** (e.g., 1000g, 2000g, 2200g)

Once answered, calculate the scaled recipe (see Recipe section below) and display it:

> **Scaled recipe — [X]g batch:**
>
> | Ingredient | Amount |
> |------------|--------|
> | AP flour (60%) | Xg |
> | Whole wheat flour (40%) | Xg |
> | Water (80%) | Xg |
> | Salt | Xg |
> | Instant yeast | X tsp |
>
> Confirm recipe, or adjust batch size?

Wait for confirmation before continuing.

---

### Step 3: Proof Method (40% Whole Wheat only)

Ask:

> **Overnight proof or same-day?**
> - **A) Same-day** — Saturday White Bread schedule: bulk ferment ~5 hrs, then 1-hour room temp final proof
> - **B) Overnight bulk** — Long overnight bulk ferment, then 1-hour room temp final proof the next morning

---

### Step 4: Start Date + Time

Ask:

> **When are you starting? (date + time)**
> e.g., "today at 9:30 AM", "tomorrow at 5:00 PM", "Apr 15 at 10:00 AM"

Parse the response into a concrete datetime. This is the **autolyse start time** — all subsequent steps are calculated from it.

Determine the EDT/EST offset for the date (`America/New_York`):
- EDT (Mar–Nov): `-04:00`
- EST (Nov–Mar): `-05:00`

---

### Step 5: Build Schedule

#### Recipe: 40% Whole Wheat Straight Dough

**Base recipe (per 1000g batch):**

| Ingredient | Amount | Baker's % |
|------------|--------|-----------|
| AP flour | 326g | 60% |
| Whole wheat flour | 217g | 40% |
| Water | 435g | 80% |
| Salt | 22g | ~4% |
| Instant yeast | ¾ tsp | — |

**Scaling formula** (scale factor `s = target_grams / 1000`):
- AP flour: `round(326 × s)`g
- WW flour: `round(217 × s)`g
- Water: `round(435 × s)`g
- Salt: `round(22 × s, 1)`g
- Yeast: `round(0.75 × s, 2)` tsp

---

#### Schedule A: Same-Day (Saturday White Bread)

All times calculated from autolyse start `T`:

| Offset | Step | Duration | Notes |
|--------|------|----------|-------|
| T + 0:00 | Autolyse | 30 min | Mix flour + water only; cover and rest |
| T + 0:30 | Mix — add yeast + salt | 15 min | Add yeast and salt; mix thoroughly |
| T + 1:00 | Fold 1 | 10 min | Stretch and fold |
| T + 1:30 | Fold 2 | 10 min | Stretch and fold |
| T + 2:00 | Fold 3 | 10 min | Stretch and fold |
| T + 2:30 | Fold 4 | 10 min | Stretch and fold; leave undisturbed for bulk ferment |
| T + 5:30 | Divide & Shape | 15 min | Bulk ferment complete (~5 hrs from mix); divide and shape loaves |
| T + 5:45 | Final Proof + Preheat Oven | 45 min | Place loaves in bannetons; preheat oven + Dutch oven to 475°F |
| T + 6:30 | Bake Covered | 30 min | Load into Dutch oven; bake covered at 475°F |
| T + 7:00 | Remove Lid — Bake Uncovered | 20 min | Remove lid; continue at 475°F until deep golden crust |
| T + 7:20 | Done | — | Remove and cool on wire rack |

---

#### Schedule B: Overnight Bulk Ferment

**Day 1** — Evening start (autolyse start = `T`):

| Offset | Step | Duration | Notes |
|--------|------|----------|-------|
| T + 0:00 | Autolyse | 30 min | Mix flour + water only; cover and rest |
| T + 0:30 | Mix — add yeast + salt | 15 min | Add yeast and salt; mix thoroughly |
| T + 1:00 | Fold 1 | 10 min | Stretch and fold |
| T + 1:30 | Fold 2 | 10 min | Stretch and fold |
| T + 2:00 | Fold 3 | 10 min | Stretch and fold |
| T + 2:30 | Fold 4 | 10 min | Stretch and fold; cover and leave at room temp overnight |
| T + 2:30 | Bulk Ferment (overnight) | ~12–14 hrs | Do not disturb; room temp ~70°F |

**Day 2** — Morning, ask for divide + shape start time, or calculate as T + 14:30 by default:

| Offset from Day 2 start `D` | Step | Duration | Notes |
|-----------------------------|------|----------|-------|
| D + 0:00 | Divide & Shape | 15 min | Bulk ferment complete; divide and shape loaves |
| D + 0:15 | Final Proof + Preheat Oven | 45 min | Place in bannetons; preheat oven + Dutch oven to 475°F |
| D + 1:00 | Bake Covered | 30 min | Load into Dutch oven; bake covered at 475°F |
| D + 1:30 | Remove Lid — Bake Uncovered | 20 min | Remove lid; continue at 475°F until deep golden crust |
| D + 1:50 | Done | — | Remove and cool on wire rack |

For Schedule B, the Day 2 start time `D`:
- If `T + 14:30` falls in a reasonable morning window (6 AM–10 AM), use it as the default.
- Otherwise, ask: *"What time do you want to divide and shape tomorrow morning?"*

---

### Step 6: Preview

Output a compact schedule table:

| Time | Step | Details |
|------|------|---------|
| 9:30 AM | Autolyse | Mix Xg AP flour + Xg WW flour + Xg water. Cover and rest 30 min. |
| 10:00 AM | Mix | Add Xg salt + X tsp yeast. Mix thoroughly. |
| ... | ... | ... |

Include the **full scaled recipe** in the description of the first event (Autolyse).

Then ask: **"Add these to your calendar?"** — wait for explicit approval.

---

### Step 7: Cleanup Previous Bread Events

Before creating new events, fetch all EA Schedule events for the target date(s) and delete any with `#bread-scheduled` in the description:

```bash
gws calendar events list --params '{
  "calendarId": "[YOUR_EA_SCHEDULE_CALENDAR_ID]",
  "timeMin": "DATE_T00:00:00OFFSET",
  "timeMax": "DATE_T23:59:59OFFSET",
  "singleEvents": true
}' 2>/dev/null
```

For overnight schedules, check both Day 1 and Day 2 dates.

---

### Step 8: Create Events

Create each event **sequentially** (not in parallel). Use this structure for each event:

```bash
gws calendar events insert \
  --params '{"calendarId": "[YOUR_EA_SCHEDULE_CALENDAR_ID]"}' \
  --json '{
    "summary": "🍞 Bread — STEP_NAME",
    "description": "STEP_DETAILS\n\nRECIPE_IF_FIRST_EVENT\n\n#bread-scheduled",
    "start": {"dateTime": "DATE_TSTART_TIMEOFFSET", "timeZone": "America/New_York"},
    "end": {"dateTime": "DATE_TEND_TIMEOFFSET", "timeZone": "America/New_York"}
  }' 2>/dev/null
echo "exit: $?"
```

**Rules:**
- All summaries prefixed with `🍞 Bread —`
- All descriptions include `#bread-scheduled` (not `#ea-scheduled`) — this is the idempotency tag
- First event (Autolyse) description includes the full scaled recipe
- Check `exit: $?` — exit 0 = success. Never retry a successful create.
- Sequential only — no parallel creates, no background subshells

---

### Step 9: Confirm

Display a compact confirmation table:

| Time | Step | Status |
|------|------|--------|
| 9:30 AM | Autolyse | Created |
| ... | ... | ... |

---

## Rules

- Never delete calendar events without `#bread-scheduled` in the description
- Always preview and wait for approval before writing to calendar
- Include the scaled recipe in the Autolyse event description
- For overnight schedules spanning two days, create events on both calendar dates
- If someone else needs to manage an oven step while James is away, note their name in the event description (ask who if unknown)
- Minimum event duration: 10 minutes (fold steps) — don't skip short steps, they're the reminders

---

## Adding New Bread Types

To add a new bread, append a new recipe block under **Step 5** with:
- Ingredient list + baker's percentages
- Scaling formula
- Named schedule(s) with offset table
- Any proof variants
