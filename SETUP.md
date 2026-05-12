# Setup & Sharing Guide — `/slack-section-brief`

**Author:** [Your Name] ([your.email@company.com])
**Last updated:** 2026-05-11

---

## What You're Installing

Two files. No code to compile, no dependencies to install beyond Claude Code + the Slack MCP plugin.

| File | Purpose | Install Location |
|---|---|---|
| `SKILL.md` | The skill definition — Claude reads this to run `/slack-section-brief` | `~/.claude/skills/slack-section-brief/SKILL.md` |
| `slack_sections.md` | Your personal channel registry — maps section names to channel IDs | `~/.claude/skills/slack-section-brief/slack_sections.md` |

---

## Prerequisites

Before you start, make sure you have:

| Requirement | How to Check |
|---|---|
| **Claude Code CLI** | Run `claude --version` — should return a version number |
| **Slack MCP plugin** | In Claude Code, run `/slack-section-brief` — if it says "Slack tools not available", the plugin isn't configured |
| **Channel membership** | You must be a member of any private 🔒 channels you want the skill to read |

---

## Step 1: Install the Skill

```bash
mkdir -p ~/.claude/skills/slack-section-brief
cp SKILL.md ~/.claude/skills/slack-section-brief/SKILL.md
```

That's it. Claude Code picks up skills from `~/.claude/skills/` automatically — no restart, no config change. The skill will appear as `/slack-section-brief` in your next Claude Code session.

---

## Step 2: Build Your Channel Registry

> ⚠️ **You cannot use the example `slack_sections.md` directly.** It reflects someone else's Slack sidebar — their sections, their channels, their private channel memberships. Your sidebar is different.
>
> **Why?** Slack sections are a purely client-side UI feature. There is no Slack API endpoint that exposes sections or the channels in them. The registry must be bootstrapped manually from screenshots of your own sidebar, then maintained as your sections change.

### How to Bootstrap

1. **Open Slack** and navigate to your sidebar. Find the first section you want to summarize.

2. **Take a screenshot** that shows all the channels in that section (scroll if needed — capture the full list).

3. **In Claude Code**, share the screenshot and say:
   ```
   Add the `_SectionName` section to slack_sections.md
   ```
   Replace `_SectionName` with the actual name of your section (e.g., `Section1`, `Section2`, `Section3`).

   Claude will:
   - Read the channel names from your screenshot
   - Resolve each channel ID via `slack_search_channels`
   - Flag any Shared/External channels (🔗) it can't resolve via API
   - Note any DMs or group DMs (excluded — not channels)
   - Append the completed section table to `slack_sections.md`

4. **Repeat** for each section you want to cover.

5. Claude places the finished file at `~/.claude/skills/slack-section-brief/slack_sections.md` by default — right alongside the skill. No extra steps needed.

### Tips for Screenshots

- Make sure all channel names are visible and readable
- If a section is long, scroll to capture all channels — the skill can only read channels it knows about
- Bold channel names in the sidebar just mean unread messages — it doesn't indicate a different channel type
- Channels with the 🔗 "External" badge are shared/external — Claude will mark these as unresolvable (see below)

### The Shared/External Channel Limitation

Channels with the **"External"** badge are cross-workspace shared channels (e.g., a joint channel with a vendor). The Slack search API cannot resolve their IDs. The skill will automatically skip them and note them in the Coverage Notes section of the brief.

**If you need to include one anyway:** Open the channel in Slack → click the channel name → "Copy link". The channel ID is the `CXXXXXXXXX` part of the URL. Add it to your registry manually with the 🔗 type flag.

---

## Step 3: Run the Skill

```
/slack-section-brief <section-name>
```

**Examples:**
```
/slack-section-brief Section1
/slack-section-brief Section7
/slack-section-brief Section5
/slack-section-brief          ← (no args) lists your available sections
```

**Matching rules:**
- Case-insensitive: `section1`, `Section1`, `SECTION1` all work
- Leading underscores ignored: `_Section1` and `Section1` match the same section

---

## Registry File Lookup

The skill searches for `slack_sections.md` in this order and uses the first file it finds:

1. `~/.claude/skills/slack-section-brief/slack_sections.md` ← **default/recommended**
2. `./slack_sections.md` (current working directory when Claude Code was launched)
3. `~/slack_sections.md` (home directory)

---

## Maintaining Your Registry

Your `slack_sections.md` needs to stay in sync with your actual Slack sidebar:

| Situation | What to Do |
|---|---|
| You joined a new channel and added it to a section | Add a row to the section table in `slack_sections.md` |
| You created a new Slack sidebar section | Screenshot it → "Add the `_NewSection` section to `slack_sections.md`" |
| A channel was renamed | Update the channel name in the table (the ID stays the same) |
| A channel was archived | Remove the row, or add an "Archived" note in the Notes column |
| A private channel became inaccessible (you left or were removed) | The skill will flag it as `⚠️ Not accessible` — remove from registry or rejoin |
| A new cron script now covers some of the channels | Add them to the Cross-Section Overlap table at the bottom of the file |

The registry doesn't auto-update — it's a one-time bootstrap + manual maintenance. In practice, sections change infrequently, so this is a low-overhead file.

---

## Sharing With a Colleague

To share the skill with someone else:

1. Send them this repo link: `[your repo or shared location]`
2. They follow Steps 1–3 above to install and bootstrap their own registry
3. Their `slack_sections.md` will be different from yours — that's expected

**What can be shared directly (same for everyone):**
- `SKILL.md` — identical install, works for any user

**What must be rebuilt per person:**
- `slack_sections.md` — must reflect the individual user's Slack sidebar sections and channel memberships

---

## Quick Reference Card

```
/slack-section-brief — Setup Checklist

□ 1. Install skill:
      mkdir -p ~/.claude/skills/slack-section-brief
      cp SKILL.md ~/.claude/skills/slack-section-brief/SKILL.md

□ 2. Build registry (one-time):
      - Screenshot each Slack sidebar section
      - Tell Claude: "Add the _SectionName section to slack_sections.md"
      - Claude resolves all channel IDs from your screenshots
      - File is saved to ~/.claude/skills/slack-section-brief/slack_sections.md

□ 3. Run:
      /slack-section-brief <section-name>
        → brief in conversation only

      /slack-section-brief <section-name> --canvas new
        → brief in conversation + creates a new Slack canvas, returns canvas ID

      /slack-section-brief <section-name> --canvas F0XXXXXXX
        → brief in conversation + replaces content of that canvas

WHY you need your own slack_sections.md:
  Slack sections = client-side only. No API exposes them.
  Your layout ≠ anyone else's layout.
  Private channel access is per-user.
```
