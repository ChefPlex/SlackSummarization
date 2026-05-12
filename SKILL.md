---
name: slack-section-brief
description: Summarize all Slack channels in a named sidebar section (as defined in slack_sections.md) over a 30-day window, with hot callouts for the last 1 day and 3 days at the top. Usage: /slack-section-brief <section-name> [--canvas <canvas-id|new>]
version: 1.2.0
author: [Your Name] <your.email@company.com>
user-invocable: true
disable-model-invocation: false
---

# Slack Section Brief Skill

Reads all channels in a named Slack sidebar section (as defined in `slack_sections.md`), summarizes activity over the last 30 days, and surfaces hot callouts for the last 24 hours and last 3 days at the top.

Optionally writes the brief directly to a Slack canvas — either replacing an existing one or creating a new one.

## Usage

```
/slack-section-brief <section-name> [--canvas <canvas-id|new>]
```

**Examples:**
- `/slack-section-brief Section1` → output to conversation only
- `/slack-section-brief Section2` → output to conversation only
- `/slack-section-brief Section3 --canvas new` → create a new canvas, write the brief to it, return the canvas URL
- `/slack-section-brief Section4 --canvas F0B1QKG7NQN` → replace the content of that canvas with the brief
- `/slack-section-brief` (no args) → list available sections and ask user to choose

Section names are **case-insensitive** and **ignore leading underscores** for matching (so `domain`, `_Domain`, and `_domain` all match `_Domain`).

---

## Execution Workflow

Follow these steps exactly when the skill is invoked.

---

### PHASE 0: Load Section Registry

Locate `slack_sections.md` by checking these paths in order, using the first one that exists:
1. `~/.claude/skills/slack-section-brief/slack_sections.md`
2. `./slack_sections.md` (current working directory)
3. `~/slack_sections.md` (home directory)

If none exist, report: "Cannot find slack_sections.md. Place it at ~/.claude/skills/slack-section-brief/slack_sections.md or in your current working directory. See the setup instructions in README.md"

Parse all `## Section:` blocks to extract:
- Section name (e.g. `Section1`, `Section2`, `Section3`)
- For each row in the section table: channel name, channel ID, type flag (🔒/🌐/🔗), notes

**Parse `$ARGUMENTS` as follows:**
- Strip any `--canvas <value>` flag from the argument string and store the value separately as `$CANVAS_ARG`
  - `--canvas new` → will create a new canvas after composing the brief
  - `--canvas F0XXXXXXX` → will replace the content of that existing canvas
  - No `--canvas` flag → output to conversation only
- The remaining argument string (after stripping `--canvas`) is the section name

**If the section name (after stripping flags) is empty:**
List all available section names and ask:
> "Which section would you like to summarize? Available sections: [list]"
Stop and wait for input.

**If the section name does not match any section** (after normalizing case and stripping leading `_`):
> "Section '[arg]' not found in slack_sections.md. Available sections: [list]"
Stop.

**Skip these channel types** — do not attempt to read them:
- 🔗 Shared/External channels (marked `⚠️ UNKNOWN` ID) — API cannot read them
- Any row with ID containing `⚠️` or `UNKNOWN`

**Note which channels are already covered by existing cron scripts** (from the "Cross-Section Overlap" table in `slack_sections.md`) — flag these in the output so the user knows the coverage already exists elsewhere.

---

### PHASE 1: Fetch Channel Messages

For each readable channel in the section, call `slack_read_channel` with:
- `limit`: 100
- `response_format`: concise

**Compute time boundaries from today's date (available in system prompt):**
- `cutoff_30d` = today minus 30 days (Unix timestamp)
- `cutoff_3d` = today minus 3 days (Unix timestamp)
- `cutoff_1d` = today minus 1 day / 24 hours (Unix timestamp)

Use the `oldest` parameter set to `cutoff_30d` to avoid fetching older messages.

**Batch the reads to avoid rate limits** — process 3 channels per batch, pause between batches:
- Batch reads together in groups of 3 using parallel tool calls
- If a channel returns an error (e.g., `channel_not_found`, `not_in_channel`), note it as "⚠️ Not accessible" and continue

After the first page, if a channel has more messages (cursor available) AND the oldest returned message is still within the 30-day window, fetch one additional page with the cursor.

---

### PHASE 2: Classify Messages by Recency

For each message fetched, classify it into one of three buckets based on its `ts` timestamp:
- **HOT-1d**: `ts >= cutoff_1d` (last 24 hours)
- **HOT-3d**: `cutoff_3d <= ts < cutoff_1d` (24 hours to 3 days ago)
- **30d**: `cutoff_30d <= ts < cutoff_3d` (3 to 30 days ago)

---

### PHASE 3: Compose the Brief

Compose the full brief content as Markdown regardless of the output destination (conversation or canvas). Always output it to the conversation as well so the user can see it.

Use this structure:

```
# [Section Name] — Slack Section Brief
> Period: Last 30 days | As of: [current date/time PDT]
> Channels: [N readable] of [N total] ([N skipped — shared/external or inaccessible])

---

## 🔥 HOT — Last 24 Hours
[Bullets for genuinely new/active items across ALL channels in the section from the last 24h]
- Lead each bullet with the substance, not "X asked about Y" — what was said, decided, or escalated
- Include: person name, channel, timestamp, specific details (org IDs, WI numbers, counts, error messages)
- Use severity indicators: 🚨 critical/blocker · ⚠️ risk/issue · 💡 notable/FYI
- Cross-reference channels if the same thread appears in multiple places
- If nothing: "No activity in the last 24 hours."

---

## 📅 HOT — Last 3 Days
[Same format, covering 24h–3d window only — do not repeat items already in the 24h section]
- If nothing: "No significant activity in the last 3 days."

---

## 📊 30-Day Summary by Channel

### [#channel-name](https://your-workspace.slack.com/archives/CXXXXXXXXX)
**[N messages in 30 days]** | [Also covered by: script-name.sh (if applicable)]

[3–7 bullets summarizing the dominant themes, decisions, recurring issues, and key people active in this channel over the 30-day window. Focus on patterns, not just individual messages. Flag anything that looks like an ongoing risk or open action item.]

[Repeat for each channel]

---

## 🧵 Cross-Channel Themes
[5–7 bullets identifying patterns, dependencies, or risks that appear across multiple channels in this section. This is the "so what" synthesis — what does the combined signal from all these channels tell you about the state of this program/domain?]

---

## 📋 Coverage Notes
- ⚠️ #channel-name — Shared/external channel, skipped (ID not resolvable via API)
- ⚠️ [#channel-name](https://your-workspace.slack.com/archives/CXXXXXXXXX) — Not accessible (not a member / private)
- ℹ️ [#channel-name](https://your-workspace.slack.com/archives/CXXXXXXXXX) — Already covered by [script-name.sh](canvas: [Canvas F0XXXXXXX](https://your-workspace.slack.com/docs/TXXXXXXXXX/F0XXXXXXX) if known)

**Inaccessible channels go here and ONLY here** — never create a `## #channel-name` section header for a channel that could not be read. Do not use `<#CXXXXXXXXX>` raw tags anywhere in Coverage Notes.
```

---

### Formatting Rules

#### References — STRICT RULES (violations degrade canvas rendering)

- **Channel headers** (section titles) use clickable Slack deep links:
  `[#channel-name](https://your-workspace.slack.com/archives/CXXXXXXXXX)`
  — inaccessible channels go in **Coverage Notes only**, never as a bare `## #channel-name` header

- **Inline channel refs** in body text MUST use canvas ref format:
  ✅ `![](#CXXXXXXXXX)`
  ❌ NEVER `<#CXXXXXXXXX>` — raw Slack mention tags do not render in canvases

- **Inline user refs** in body text MUST use canvas ref format:
  ✅ `![](@UXXXXXXX)`
  ❌ NEVER `<@UXXXXXXX>` — raw Slack mention tags do not render in canvases
  — This applies everywhere: HOT bullets, channel summaries, `cc` lines, all body text

- **GUS work items** always linked:
  `[W-XXXXXXX](https://gus.lightning.force.com/lightning/r/ADM_Work__c/W-XXXXXXX/view)`
  — Never mention a W-number as plain text

- **Canvas IDs** mentioned in body text always linked:
  `[Canvas F0XXXXXXX](https://your-workspace.slack.com/docs/TXXXXXXXXX/F0XXXXXXX)`
  — Never mention a canvas ID (F0...) as plain text

- **PRs, ERRs, MRs, KAs, any linked artifact**: always include the URL — never mention without a link

#### HOT Section Format

HOT bullets lead with the **substance** (what was decided, escalated, or changed) — not with the person's name. Person and timestamp are secondary context:

✅ `- ⚠️ Cold-start fix plan published — deploy PR #911 to prod by May 12, full prod May 14 in report-only mode (Virender Singh, 11:12 AM, ![](#C06KA4ENG3S))`
❌ `- 🟡 **#inline-detections | Virender Singh @ 11:12 AM** — Published cold-start fix plan...`

Severity indicators are required on every HOT bullet:
- 🚨 critical/active blocker
- ⚠️ risk, issue, or decision needed
- 💡 notable/FYI

#### Canvas Self-Link (when writing to a canvas)

When the output target is a canvas (via `--canvas`), add a self-referential link as the second line of the canvas, immediately after the title. This lets readers copy and share the canvas URL:

```
> 📎 [This canvas](https://your-workspace.slack.com/docs/TXXXXXXXXX/F0XXXXXXX) | Last updated: [date/time PDT]
```

Replace `F0XXXXXXX` with the actual canvas ID being written to.

#### General

- **Bullets over paragraphs** — dense, scannable, no filler prose
- **DO NOT dumb down the content** — include specific technical details: exact PR numbers, GUS IDs, user names, timestamps, detection counts, error messages. A vague summary is worse than useless.
- **Only report what actually happened** — do not fabricate or carry forward stale info as new activity

---

### Content Quality Rules

- **HOT sections**: Only include items that are genuinely time-sensitive or actionable. Don't pad with routine noise.
- **30-day per-channel summaries**: Identify the 3–5 most important threads or patterns. What is this channel actually being used for right now? What's the dominant concern?
- **Cross-channel themes**: This is the highest-value output. Connect the dots — what do these channels collectively signal about program health, risk posture, or upcoming decisions?
- **Stale channels**: If a channel had zero messages in the 30-day window, say: "No activity in the last 30 days." in one line. Don't pad.
- **RAG status**: If you can infer a RAG status for the section's overall health from the signal, include it at the top: 🟢 GREEN / 🟡 YELLOW / 🔴 RED with a one-sentence rationale.

---

### PHASE 3.5: Write to Canvas (if `--canvas` was specified)

**If `$CANVAS_ARG` is `new`:**
1. Call `slack_create_canvas` with:
   - `title`: `[Section Name] — Slack Section Brief`
   - `content`: the full brief Markdown composed in PHASE 3, with the self-link line inserted as the second line:
     `> 📎 [This canvas](https://your-workspace.slack.com/docs/TXXXXXXXXX/[canvas-id]) | Last updated: [date/time PDT]`
     (use the canvas ID returned by `slack_create_canvas`)
2. Report the new canvas ID and URL to the user:
   > "✅ Canvas created: [Section Name] — Slack Section Brief  
   > URL: https://your-workspace.slack.com/docs/TXXXXXXXXX/[canvas-id]  
   > Canvas ID: [canvas-id] — save this to reuse with `--canvas [canvas-id]`"

**If `$CANVAS_ARG` is a canvas ID (starts with `F`):**
1. Prepend the self-link as the second line of the brief content:
   `> 📎 [This canvas](https://your-workspace.slack.com/docs/TXXXXXXXXX/[canvas-id]) | Last updated: [date/time PDT]`
2. Call `slack_update_canvas` with:
   - `canvas_id`: `$CANVAS_ARG`
   - `action`: `replace`
   - `content`: the full brief Markdown with self-link included
3. Confirm to the user:
   > "✅ Canvas updated: https://your-workspace.slack.com/docs/TXXXXXXXXX/[canvas-id]"

**If canvas write fails:**
Report the error clearly — do not silently skip. The conversation output is still valid.

---

### PHASE 4: Offer Next Steps

After the brief, offer:

```
---
**What next?**
- Save this brief to a new canvas → rerun with `--canvas new`
- Update an existing canvas → rerun with `--canvas <canvas-id>`
- Drill into a specific channel → say "deep dive on #channel-name"
- Build a cron script for this section → say "build a cron for [section-name]"
- Set up a scheduled brief → say "set up [section-name] as a scheduled brief"
```

(If `--canvas` was already used, omit the canvas suggestions from the next-steps prompt.)

---

## Error Handling

| Situation | Action |
|---|---|
| `slack_sections.md` not found | "Cannot find slack_sections.md. Place it at ~/.claude/skills/slack-section-brief/slack_sections.md or in your current working directory. See README.md for setup instructions." |
| Section not in registry | List available sections and stop |
| All channels inaccessible | Report each failure, explain likely cause (private channel, bot not added) |
| Channel returns 0 messages in 30d | State "No activity in the last 30 days" — don't skip silently |
| Shared/external channel | Skip read attempt, note in Coverage Notes |
| Rate limit hit | Wait and retry; note in output if retries exhausted |
| `--canvas new` fails | Report the error; output is still valid in the conversation |
| `--canvas F0XXXXXXX` not found | "Canvas [id] not found or not accessible. Brief is available in the conversation." |
| `--canvas` value is neither `new` nor starts with `F` | "Unrecognized --canvas value '[value]'. Use `--canvas new` to create a canvas or `--canvas F0XXXXXXX` with an existing canvas ID." |
