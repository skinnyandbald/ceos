---
name: ceos-scorecard-autopull
description: "Enhanced scorecard entry with auto-pull from L10 meetings and available MCP sources (CRM, calendar, email, filesystem). Walks through each metric one at a time with pre-populated suggestions for interactive confirmation."
file-access: [data/scorecard/, data/meetings/l10/]
tools-used: [Read, Write, Glob, Grep, Bash, AskUserQuestion]
---

# ceos-scorecard-autopull

Enhanced weekly scorecard entry that synthesizes data from **L10 meetings** and any available MCP sources (CRM, calendar, email, filesystem). Walks through each metric **one at a time** with a pre-populated suggestion, letting the user confirm or tweak before proceeding to the next metric.

**This skill wraps `ceos-scorecard` -- it does NOT replace it.** After interactive confirmation, it writes the file using the same format as `ceos-scorecard` Mode: Log Weekly.

## When to Use

Use this anytime you want to log scorecard numbers with auto-populated suggestions instead of entering them manually.

## Context

### Finding the CEOS Repository

Search upward from the current directory for the `.ceos` marker file. This file marks the root of the CEOS repository. If the project's CLAUDE.md specifies a CEOS/EOS root path, use that instead.

If `.ceos` is not found, stop and tell the user: "Not in a CEOS repository. Clone your CEOS repo and run setup.sh first."

### Key Files (all relative to CEOS root)

| File | Purpose |
|------|---------|
| `data/scorecard/metrics.md` | Metric definitions (names, owners, goals, thresholds) |
| `data/scorecard/weeks/YYYY-WNN.md` | Weekly scorecard entries |
| `data/meetings/l10/` | L10 meeting notes (primary data source) |

## Process

### Step 1: Determine the Week

Calculate the current ISO week (`YYYY-WNN`). Check if `data/scorecard/weeks/YYYY-WNN.md` already exists. If so, tell the user and ask if they want to update it.

Determine the week's date range (Monday through Sunday) for filtering queries.

### Step 2: Read Metric Definitions

Read `data/scorecard/metrics.md` to get the list of metrics, their goals, and green/red thresholds.

### Step 3: Synthesize from L10 Meetings (PRIMARY SOURCE)

**This is the most important data source.** Read all L10 meeting files from `data/meetings/l10/` that fall within the scorecard week's date range.

For each L10 file in the week:
1. Read the full file
2. Extract from **Headlines** section: deal closings, revenue mentions, pipeline movement, content published, milestones hit
3. Extract from **Scorecard Review** section: any metric values already discussed
4. Extract from **IDS** section: context about what's working/not working
5. Extract from **To-Do Review** section: completed actions that map to metrics
6. Extract from **Conclude / New To-Dos** section: commitments that indicate activity

**Build a synthesis per metric** by scanning all L10s for the week. Match L10 content to each metric based on the metric's name and description from `metrics.md`.

### Step 4: Auto-Pull from Available MCP Sources (optional, run in parallel)

**Discover available MCP tools** at runtime. Check which MCP servers are connected and query them for supplementary data. This step is entirely optional — the skill works with L10 data alone.

Common MCP sources and what to query:

| Source Type | Example Tools | What to Query |
|-------------|---------------|---------------|
| CRM (Attio, HubSpot, etc.) | `search_records`, `search_records_advanced` | Deal stage changes, pipeline value, revenue |
| Calendar (Google, Outlook) | `list_events`, `gcal_list_events` | Meetings booked, discovery calls |
| Email (Gmail, etc.) | `search_messages` | Correspondence related to deals |
| Filesystem | Glob, Grep | Published content files, shipped deliverables |

**If a CLAUDE.md specifies paths for content, episodes, or other deliverables**, scan those paths for files with date prefixes matching the scorecard week.

**If no MCP sources are available, skip this step entirely.** The L10 synthesis from Step 3 is sufficient.

### Step 5: Merge Sources and Resolve Conflicts

For each metric, merge the L10 synthesis with any external source data:

- **L10 says X, external source says Y** -- Present both, flag the discrepancy, let the user decide.
- **L10 says X, no external data** -- Use L10 value as the suggestion.
- **No L10 data, external source says Y** -- Use external value as the suggestion.
- **Neither source has data** -- Suggest 0, note "no data found -- manual entry needed."

**Priority order:** User's words in L10s > CRM data > Calendar data > Filesystem > Default to 0.

### Step 6: Walk Through Each Metric (Sequential Questions)

**This is the core interaction.** Present ONE metric at a time using `AskUserQuestion`. Each question shows:
- The metric name and goal
- The synthesized suggestion with evidence
- The source(s) it came from
- Status based on the suggested value

Walk through all metrics in the order defined in `metrics.md`. For each metric:

1. Show a brief summary of what was found:
   ```
   ### [Metric Name] (Goal: [goal]/week)

   Suggested: [value]
   Sources:
   - [Day] L10: "[relevant quote or data point]"
   - [External source]: "[relevant data]"

   Status: [on_track/off_track] (goal: [goal], red threshold: [threshold])
   ```

2. Ask using `AskUserQuestion` with the suggested value as the default option:
   - Option 1: "[value] (suggested)" -- accept the synthesized number
   - Option 2: "Different number" -- user provides the correct number
   - Option 3 (if applicable): "0 -- didn't track" -- explicit zero

3. Record the confirmed value before moving to the next metric.

**Key principle:** The pre-populated suggestion should save the user time. If the L10s captured good data, the user just confirms. If not, they override. Either way, they only deal with one metric at a time.

### Step 7: Compile Notes

After all metrics are confirmed, auto-generate the Notes section:

For each metric, create a note line:
- `**[Metric]: [Value] ([status])** -- [source/context from L10s and external data]`

Include a "L10 Context" subsection summarizing key themes from the week's L10s (IDS issues tackled, major headlines, patterns).

If external sources were used, include a "Data Sources" line listing what was queried.

### Step 8: Show Final Scorecard and Write

Present the complete scorecard table:

```
| Metric | Owner | Goal | Actual | Status |
|--------|-------|------|--------|--------|
| ... | ... | ... | ... | ... |
```

Ask: "Save this week's scorecard?"

On confirmation, write to `data/scorecard/weeks/YYYY-WNN.md` using the standard CEOS template format:

```markdown
---
week: "YYYY-WNN"
date: "YYYY-MM-DD"
---

# Scorecard -- YYYY-WNN

*Logged: YYYY-MM-DD (date range)*

| Metric | Owner | Goal | Actual | Status |
|--------|-------|------|--------|--------|
| ... |

## Notes

- **Metric: Value (status)** -- context
...

## L10 Context (synthesized from meetings)

Key themes this week:
- [theme from IDS discussions]
- [pattern across multiple L10s]

## Data Sources

*Synthesized from: L10 meetings ([N] this week)[, CRM (deals/stages)][, Calendar (meetings)][, filesystem (content/deliverables)].*
```

### Step 9: Flag Off-Track Items

For any off_track metric, note: "Off-track items should be discussed during your next L10."

If 3+ metrics are off track, add: "Consider whether this week was an anomaly or a pattern worth investigating."

Offer to commit the scorecard file.

## Guardrails

- **Never write the scorecard without user confirmation.** The whole point is interactive review.
- **One metric at a time.** Don't dump all metrics in one question. Walk through them sequentially so the user can focus.
- **Show sources for every suggestion.** Don't just show a number -- show where it came from (which L10, which data source).
- **L10s are the primary source.** They capture what the user actually said happened. External sources are validation/supplementation.
- **Graceful degradation.** If no L10s exist for the week, fall back to external sources only. If no MCP sources are available, ask for manual entry. Don't fail the whole flow.
- **All paths relative to CEOS root.** Never hardcode absolute paths. Use `data/scorecard/weeks/`, `data/meetings/l10/`, etc.
- **Respect metric definitions.** Use thresholds from `metrics.md` for status calculation.
- **Keep it conversational.** This runs in an interactive session. Talk to the user, don't just dump data.

## Integration

- Use instead of `ceos-scorecard` when you want auto-populated suggestions for the "Log Weekly" mode
- `ceos-scorecard` is still available for "Define Metrics" and "Trend Analysis" modes
- After writing, suggest running this week's L10 if one hasn't been done yet
