# SlackSummarization

The original Slack channel summarization skill - built to solve a real problem in a real program environment before it became something shareable.

This tool reads Slack channels on demand and surfaces what matters: recent activity, hot items, decisions, risks, and cross-channel patterns. It was built to replace the 20-minute manual process of checking five channels before writing a status update, and it worked.

---

## What This Is

This is the original version of the work that eventually became the Slack Crawler and Slack Canvas Status Synthesis tools in [tpm-toolbox](https://github.com/ChefPlex/tpm-toolbox). It was built for a specific internal environment and reflects that context - the structure, the section names, and some of the operational assumptions are specific to where it was used.

It is published here as a record of the original work. The general-use versions, cleaned up and documented for use outside any one organization, live in tpm-toolbox.

---

## What It Does

- Reads all channels in a named Slack sidebar section
- Produces a structured intelligence brief covering recent activity
- Surfaces hot callouts for the last 24 hours and 72 hours at the top
- Summarizes each channel with dominant themes, decisions, open risks, and key people
- Synthesizes cross-channel patterns - the things you would only see if you were reading everything
- Optionally writes the brief to a Slack canvas and keeps it updated

The core insight behind the tool: Slack sections are invisible to the API. There is no endpoint that tells you what channels are in a sidebar section. The solution is a one-time registry file that maps section names to channel IDs, which the skill reads at runtime. Build the registry once from screenshots of your sidebar. After that it runs entirely via API.

---

## How It Evolved

The original version was tightly coupled to a specific organizational context - section names, channel IDs, and output formats that matched one program environment.

The generalized versions extract that context into a user-maintained registry file (`slack_sections.md`) that anyone can build for their own Slack workspace in about 10 minutes. The output format was also restructured to be configurable rather than hardcoded.

**For the general-use version:** [tpm-toolbox/slack-crawler](https://github.com/ChefPlex/tpm-toolbox/tree/main/slack-crawler)

**For the status report generation layer:** [tpm-toolbox/slack-canvas-status-synthesis](https://github.com/ChefPlex/tpm-toolbox/tree/main/slack-canvas-status-synthesis)

---

## Requirements

- Claude Code CLI installed and authenticated
- Slack MCP plugin configured in Claude Code
- Membership in the private channels you want to read
- A `slack_sections.md` registry file mapping your sidebar sections to channel IDs

See the [Slack Crawler setup guide](https://github.com/ChefPlex/tpm-toolbox/blob/main/slack-crawler/SETUP.md) in tpm-toolbox for full installation and registry-building instructions.

---

## Status

This repo is not actively maintained. It reflects the state of the work at the point it was generalized and moved. Use the tpm-toolbox versions for anything new.

---

*Part of the [ChefPlex](https://github.com/ChefPlex) portfolio. Built by [Eric White](https://www.linkedin.com/in/edwhite).*
