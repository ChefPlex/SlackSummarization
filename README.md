# slack-section-brief

**A Claude Code skill that summarizes your Slack sidebar sections on demand by section as often as you want and is able to create or update a canvas as part of the process.**

Give it a section name. Get a dense, scannable intelligence brief covering the last 30 days — with hot callouts for the last 24 hours and 3 days at the top.

---

## What It Does

Slack lets you group channels into named sidebar sections (Section1, Section2, Section3, etc.). This skill reads all the channels in a named section and produces a structured brief:

- 🔥 **HOT — Last 24 Hours**: Time-sensitive items across all channels, with severity indicators
- 📅 **HOT — Last 3 Days**: Notable items from the 24h–3d window
- 📊 **30-Day Summary by Channel**: 3–7 bullets per channel — dominant themes, decisions, open risks (you can adjust this period within the skill or have Claude do it)
- 🧵 **Cross-Channel Themes**: The "so what" synthesis — what does the combined signal tell you about program health?
- 📋 **Coverage Notes**: What was skipped and why (shared channels, inaccessible private channels, channels already covered by cron jobs)

Output goes directly in your Claude Code conversation. You can optionally ask it to post the brief to a Slack canvas afterward.

---

## Why This Exists: The Slack Sections API Problem

**Slack sections are invisible to the Slack API.** There is no endpoint that lists your sidebar sections or the channels in them. Sections are a purely client-side UI feature — they exist only inside your Slack app. This means:

- You can't query "what channels are in my _Domain section?"
- You can't automate summarization by section name without first building a registry
- Every user has a different sidebar layout — sections, channels, and private channel memberships all differ per person

**The solution is a one-time bootstrap:** Take screenshots of your Slack sidebar sections, share them with Claude, and Claude resolves all the channel IDs via the Slack API. The result is saved to `slack_sections.md` — a persistent registry file that the skill reads at runtime.

Once bootstrapped, the skill works entirely via API. The registry only needs updating when your sidebar changes.

---

## Repo Contents

```
slack_summarization/
├── README.md              ← You are here
├── SKILL.md               ← The Claude Code skill definition (install this — same for everyone)
├── slack_sections.md      ← Example channel registry (reference only — build your own)
└── SETUP.md               ← Detailed setup and sharing guide
```

---

## ⚠️ What You Need to Personalize

**`SKILL.md` is the same for everyone — install it as-is.** No edits required for most users.

**`slack_sections.md` must be built fresh for every person.** The file in this repo is an example registry. It will not work for you because:
- Your Slack sidebar sections have different names
- Your sections contain different channels
- Private channels you're not a member of cannot be read — and the example registry's private channels are not your private channels

No edits to `SKILL.md` are required — the repo version has no hardcoded paths.

---

## Quick Start

### Step 1 — Install the Skill (one command)

```bash
mkdir -p ~/.claude/skills/slack-section-brief && \
cp SKILL.md ~/.claude/skills/slack-section-brief/SKILL.md
```

Claude Code picks up skills from `~/.claude/skills/` automatically. No restart needed — the skill appears as `/slack-section-brief` immediately.

---

### Step 2 — Build Your `slack_sections.md` Registry

This is the setup step unique to you. It takes 5–10 minutes and only needs to be done once.

#### What you're building

`slack_sections.md` is a Markdown file that maps each of your Slack sidebar sections to a table of channel names and IDs. It looks like this:

```markdown
# Slack Sections — Channel Registry

This file maps Slack sidebar sections to channel IDs for use with /slack-section-brief.

**Author:** Your Name (your.email@company.com)
**Last updated:** YYYY-MM-DD
**Sections documented:** `Section1`, `Section2`

---

## Section: `_SectionA`

What this section is for.

| Channel | ID | Type | Notes |
|---|---|---|---|
| #channel-one   | C0123456789 | 🔒 Private         | Notes about this channel        |
| #channel-two   | C0987654321 | 🌐 Public          | Notes about this channel        |
| #vendor-collab | ⚠️ UNKNOWN  | 🔗 Shared/External | Cannot be read via Slack API    |

**Total channels:** 3 (2 resolved, 1 unresolved)
```

The skill reads this file at runtime to know which channels to fetch for each section. Without it, the skill can't run.

#### How to build it

**For each Slack sidebar section you want to cover:**

1. **Open Slack** and find the sidebar section. Make sure all channels in the section are visible.

2. **Take a screenshot** of the section showing the complete channel list. If the section is long, scroll to capture everything — Claude can only register channels it can see in the screenshot.
   > **Tip:** Bold channel names in the sidebar just mean unread messages — not a different channel type. Channels with a small external badge (🔗) are Shared/External channels and cannot be read via API (more on this below).

3. **In Claude Code**, share the screenshot and say exactly:
   ```
   Add the `_SectionName` section to slack_sections.md
   ```
   Replace `_SectionName` with the actual name of your section (e.g., `_Domain`, `STPM`, `PlatformDefenseAndSafety`).

4. **Claude will:**
   - Read all channel names visible in your screenshot
   - Call `slack_search_channels` for each one to resolve the channel ID
   - Flag any channels it can't resolve (Shared/External, or truly unknown names)
   - Note any DMs or group DMs it found and skip them (not channels)
   - Append the completed section table to `~/.claude/skills/slack-section-brief/slack_sections.md`, creating the file if it doesn't exist

5. **Repeat steps 1–4** for each section you want to cover.

#### Where to put the file

The skill searches these paths in order and uses the first one it finds:

1. `~/.claude/skills/slack-section-brief/slack_sections.md` ← **recommended — right next to the skill**
2. `./slack_sections.md` (current working directory when Claude Code launched)
3. `~/slack_sections.md` (home directory)

Placing it alongside the skill at path 1 is simplest — everything lives in one place.

#### What to expect during ID resolution

When Claude processes your screenshot, it runs `slack_search_channels` for each channel name. Most channels resolve immediately. A few common situations where they don't:

| Situation | What Claude does | What you see in the registry |
|---|---|---|
| **Shared/External channel** ("External" badge) | Marks as unresolvable — the Slack search API cannot index cross-workspace channels | `⚠️ UNKNOWN` in the ID column, 🔗 in the Type column |
| **Private channel with an unusual name** | May not resolve if search can't find it | Claude flags it — you can provide the ID manually |
| **DMs / Group DMs** | Skipped entirely — not channels | Noted in the section footer, not added to the table |

#### Handling Shared/External channels

Channels with the **"External" badge** are cross-workspace shared channels (e.g., a joint channel with a vendor). The Slack search API cannot find them by name, so Claude cannot auto-resolve their IDs. The skill skips them automatically and notes them in the Coverage Notes section of every brief.

**If you need to include one:** Get the channel ID manually:
1. Open the channel in Slack
2. Click the channel name at the top → "Copy link"
3. The ID is the `CXXXXXXXXX` part: `https://your-workspace.slack.com/archives/`**`C0A7FBE4LJF`**
4. Edit `slack_sections.md` and replace `⚠️ UNKNOWN` with the real ID

#### A complete example

Here's what a finished `slack_sections.md` looks like with two sections documented:

```markdown
# Slack Sections — Channel Registry

This file maps Slack sidebar sections to channel IDs for use with /slack-section-brief.

**Author:** Your Name (your.email@company.com)
**Last updated:** 2026-05-11
**Sections documented:** `_Engineering`, `STPM`

---

## Section: `_Engineering`

Core engineering program channels.

| Channel | ID | Type | Notes |
|---|---|---|---|
| #eng-standup       | C0123456789 | 🌐 Public          | Daily standup coordination                      |
| #eng-incidents     | C0234567890 | 🔒 Private         | P0/P1 incident response                         |
| #eng-vendor-collab | ⚠️ UNKNOWN  | 🔗 Shared/External | "External" badge — not API-readable  |

**Total channels:** 3 (2 resolved, 1 unresolved)

---

## Section: `STPM`

Security TPM team coordination channels.

| Channel | ID | Type | Notes |
|---|---|---|---|
| #sec-tpm-team      | C0345678901 | 🔒 Private | TPM team collaboration. Creator: Jane Smith. |
| #tpm-community     | C0456789012 | 🌐 Public  | TPM best practices and learning community.   |

**Total channels:** 2 (2 resolved)

---

## Cross-Section Overlap (Existing Cron Coverage)

Channels already covered by automated cron scripts — the skill flags these in Coverage Notes
so you know the coverage already exists elsewhere.

| Channel | ID | Covered By |
|---|---|---|
| #eng-incidents | C0234567890 | `my_incidents_cron.sh` |
```

---

### Step 3 — Run It

```
/slack-section-brief <section-name> [--canvas <canvas-id|new>]
```

**Output to conversation only (default):**
```
/slack-section-brief Section1
/slack-section-brief Section7
/slack-section-brief Section5
/slack-section-brief                            ← lists all available sections
```

**Create a new Slack canvas and write the brief to it:**
```
/slack-section-brief Section7 --canvas new
```
Returns the new canvas ID and URL. Save the ID — use it on future runs to keep the same canvas refreshed.

**Replace the content of an existing canvas:**
```
/slack-section-brief Section7 --canvas F0B1QKG7NQN
```
Overwrites the canvas with a fresh brief. The brief is also shown in the conversation.

**Matching rules:**
- Section names are case-insensitive: `stpm`, `STPM`, `Stpm` all work
- Leading underscores are ignored: `_Domain` and `Domain` match the same section

---

## Registry File Lookup Order

The skill searches these paths in order and uses the first one it finds:

1. `~/.claude/skills/slack-section-brief/slack_sections.md` ← **recommended**
2. `./slack_sections.md` (current working directory when Claude Code launched)
3. `~/slack_sections.md` (home directory)

Placing the file alongside the skill at path 1 is the simplest setup — everything lives in one place and there's nothing else to configure.

---

## Prerequisites

| Requirement | Details |
|---|---|
| **Claude Code CLI** | Installed and authenticated — run `claude --version` to verify |
| **Slack MCP plugin** | Configured in Claude Code — the skill calls `slack_read_channel` internally. If this isn't set up, the skill will fail when trying to fetch messages. |
| **Channel membership** | You must be a member of any private 🔒 channels you want the skill to read |
| **`slack_sections.md`** | Must exist at one of the 3 lookup paths — see Step 2 |

---

## Output Format

```
# [Section Name] — Slack Section Brief
> Period: Last 30 days | As of: [date/time PDT]
> Channels: [N readable] of [N total] ([N skipped])
> Overall: 🟢 GREEN / 🟡 YELLOW / 🔴 RED — one-sentence rationale

---

## 🔥 HOT — Last 24 Hours
- 🚨 [critical/blocker] — person, #channel, timestamp, specific details
- ⚠️ [risk/issue] — ...
- 💡 [notable/FYI] — ...

## 📅 HOT — Last 3 Days
- [same format, 24h–3d window only, no repeats from above]

## 📊 30-Day Summary by Channel

### [#channel-name](https://your-workspace.slack.com/archives/CXXXXXXXXX)
**[N messages in 30 days]** | [Also covered by: script.sh if applicable]
- Dominant theme or decision
- Recurring issue or open risk
- ...

## 🧵 Cross-Channel Themes
- Pattern or risk that spans multiple channels
- ...

## 📋 Coverage Notes
- ⚠️ #channel-name — Shared/external channel, skipped (ID not resolvable via API)
- ⚠️ #channel-name — Not accessible (not a member)
- ℹ️ #channel-name — Already covered by my_cron.sh
```

---

## Maintenance

The registry needs to stay in sync with your actual Slack sidebar:

| Situation | Action |
|---|---|
| You joined a new channel and added it to a section | Add a row to that section's table in `slack_sections.md` — ask Claude for the ID if needed |
| You created a new sidebar section | Screenshot it → "Add the `_NewSection` section to `slack_sections.md`" |
| A channel was renamed | Update the name in the table (the channel ID doesn't change) |
| A channel was archived | Remove the row or add "Archived" to the Notes column |
| A private channel became inaccessible | Skill flags `⚠️ Not accessible` — remove from registry or rejoin the channel |
| A new cron script now covers some channels | Add those channels to the Cross-Section Overlap table |

---

## Extending This Pattern

- **Write to a canvas inline** → Add `--canvas new` or `--canvas F0XXXXXXX` directly to the command. No follow-up needed.
- **Keep a standing canvas for a section** → Run with `--canvas new` once to create it, note the canvas ID, then use `--canvas F0XXXXXXX` on every subsequent run to refresh it.
- **Build a cron script for a section** → After running a brief, say "build a cron for [section-name]". Claude generates a bash cron script + canvas update prompt modeled on the existing `edv_canvas_cron.sh` pattern.
- **Deep dive one channel** → Say "deep dive on #channel-name". Claude re-reads with a higher message limit and produces a thread-by-thread breakdown.

---

## Files in This Repo

### `SKILL.md`
The Claude Code skill definition. Install at `~/.claude/skills/slack-section-brief/SKILL.md`. Same for every user — no edits required unless you're changing the registry lookup path.

### `slack_sections.md`
Example personal channel registry. **Reference and example only.** Shows the exact format and structure — use it as a template when building yours.

### `SETUP.md`
Full setup and sharing guide with a quick-reference checklist card at the end, suitable for sending to a colleague alongside `SKILL.md`.

---

**Version:** 1.2.0
