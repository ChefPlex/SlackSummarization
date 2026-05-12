# Slack Sections — Channel Registry

This file documents your Slack sidebar sections with full channel names, IDs, types, and notes.
Use this as the authoritative source when building canvas cron scripts or summarization prompts.

**How to use in a cron script:**
- Copy the channel IDs from the section you want to monitor
- Use `slack_read_channel` (not `slack_search_public`) with `limit=50` and `response_format=concise`
- Batch reads to avoid rate limits (2-3 channels per batch)
- Private channels (🔒) are accessible via API if you are a member
- Shared/external channels (🔗) tagged "External" are NOT indexed by the Slack API — IDs unresolvable via search
- DMs and group DMs are excluded from this registry (not channels)

**Author:** [Your Name] ([your.email@company.com])
**Last updated:** 2026-05-10
**Sections documented:** `Section1`, `Section2`, `Section3`, `Section4`, `Section5`, `Section6`, `Section7`, `Section8`

---

## Section: `Section1`

Email domain verification program channels. All 7 channels are already monitored by `edv_canvas_cron.sh`.

| Channel | ID | Type | Notes |
|---|---|---|---|
| #edv-command-center | C0B0LGJ7WE9 | 🔒 Private | 24/7 command center for live rollout support. Also in `edv_canvas_cron.sh`. |
| #edv-comms-risk-treatment | C0AEBQM3FG8 | 🔒 Private | Comms workstream. Status updates posted here. Also in `edv_canvas_cron.sh`. |
| #email-domain-verification-dev-support | C0AFTGACHRV | 🌐 Public | Escalation channel; Channel Expert bot triages. Also in `edv_canvas_cron.sh`. |
| #guardian-h0-comms-enablement | C0AJW5GLGM9 | 🔒 Private | Guardian H0 comms & enablement working group. Also in `edv_canvas_cron.sh`. |
| #sec-risk-treatment-email-abuse | C09RJKBCT0A | 🔒 Private | Primary program status channel. Weekly status posted here. Also in `edv_canvas_cron.sh`. |
| #temp-domain-verification-mandate | C0AEB4GH37H | 🌐 Public | High-volume support triage. Channel Expert bot. Also in `edv_canvas_cron.sh`. |
| #temp-edv-lap-execution | C0AKNC0440N | 🔒 Private | Extension/exception execution — LAP requests, BT tooling. Also in `edv_canvas_cron.sh`. |

**Total channels:** 7 (7 resolved) — fully covered by `edv_canvas_cron.sh` → canvas `F0B1SV1EV0A`

---

## Section: `Section2`

AI tools, internal AI community, and learning channels.

| Channel | ID | Type | Notes |
|---|---|---|---|
| #ai-tools-general | C058L05637W | 🌐 Public | Company-wide AI learning hangout. |
| #ai-reporting | C0B1H5H3H54 | 🔒 Private | AI reporting channel. |
| #ai-initiatives-broadcast | C07NPSLD1S5 | 🌐 Public | Announcements for AI initiatives. |
| #ai-pilot-feedback | C0AKLTZVDMZ | 🌐 Public | AI pilot feedback channel. |

**Total channels:** 4 (4 resolved)

---

## Section: `Section3`

Operations, detect-and-respond, release planning, and V2MOM. Mostly DMs + a few channels.

| Channel | ID | Type | Notes |
|---|---|---|---|
| #ops-release-planning | C0903V29JPM | 🔒 Private | Release planning coordination. |
| #ops-v2mom-status | C04383N2XC7 | 🌐 Public | V2MOM status updates for all measures. |
| #data-notebook-help | C0448QY79CY | 🌐 Public | Support channel for internal notebook tooling. Office hours Fridays 9:30-10:30 PT. |

**Total channels:** 3 (3 resolved) — remainder of section entries are DMs/group DMs (excluded)

---

## Section: `Section4`

Security leadership context — org-wide announcements and integrated threat management.

| Channel | ID | Type | Notes |
|---|---|---|---|
| #sec-org-announcements | C03165ASU9M | 🔒 Private | Security org-wide announcements: new hires, promotions, milestones, releases. |
| #sec-threat-management | C0467DQRM70 | 🔒 Private | Integrated threat management team channel. |

**Total channels:** 2 (2 resolved) — `Doug Miller (Security)` is a DM (excluded)

---

## Section: `Section5`

Platform abuse, fraud prevention, vendor integrations, and defense & safety team channels.

| Channel | ID | Type | Notes |
|---|---|---|---|
| #vendor-fraud-prevention-internal | C06PK4MPW6R | 🔒 Private | Fraud prevention vendor implementation. |
| #core-product-anti-fraud-working-group | C079V5UB62F | 🔒 Private | Core Product Anti-Fraud Working Group. Cross-functional: Security, Product Eng, Finance, BT. |
| #platform-abuse-collab | C0451UWJ0AK | 🔒 Private | General platform abuse response ops. Cross-posting of abuse prevention updates. |
| #ext-vendor-fraud-collab | ⚠️ UNKNOWN | 🔗 Shared/External | External badge — shared channel with fraud prevention vendor. Not indexed by Slack API. |
| #mc-product-anti-fraud-working-group | C06MB4HRCSV | 🔒 Private | Marketing Cloud Product Anti-Fraud Working Group. Cross-functional: Security, Product Eng, Product Marketing. |
| #platform-abuse-corp-communications | C06Q57QREQN | 🔒 Private | Public escalations of platform abuse (social media). Triage between external comms and Security/Product. |
| #platform-defense-and-safety | C0644LK2A0G | 🔒 Private | Core platform defense and safety team channel. |
| #platform-defense-safety-public | C08BZCYU119 | 🌐 Public | Public-facing channel for RFIs and general enquiries. |
| #proj-forms-trials | C08BKN542UE | 🔒 Private | Forms/trials project channel. |
| #sec-ip-reputation-integration | C07N5P6JVQ9 | 🔒 Private | IP reputation vendor integration. Also monitored in `security_channels_brief_cron.sh`. |
| #security-industry-news | C02N2CEU0TD | 🌐 Public | Security news, vulnerabilities, attacks, tools, research articles. |
| #starter-fraud | C07L8B94U1K | 🔒 Private | Starter fraud workstream. |
| #temp-comms-email-domain-verification | C0ADJL45L66 | 🔒 Private | Temp comms channel for email domain verification. |
| #temp-vpn-proposal | C098USYGE13 | 🔒 Private | VPN proposal scoped temp channel. |

**Total channels:** 14 (13 resolved, 1 unresolved — external vendor channel is shared/external)

---

## Section: `Section6`

Sensitive/restricted channels. Minimal channels — mostly DMs.

| Channel | ID | Type | Notes |
|---|---|---|---|
| #sec-restricted-ceremony | C06KX1WKQQY | 🔒 Private | Security restricted ceremony channel. |

**Total channels:** 1 (1 resolved) — remainder are DMs/group DMs (excluded)

---

## Section: `Section7`

Security TPM team channels — team coordination, V2MOM, governance, and TPM community.

| Channel | ID | Type | Notes |
|---|---|---|---|
| #org-all-announcements | C01H5DGGQPJ | 🌐 Public | Org-wide announcements channel. |
| #sec-local-social | C6244QEBX | 🔒 Private | Local security team social/coordination channel. |
| #sec-tpm-team | C077U8W5WF5 | 🔒 Private | Security TPM team collaboration space. |
| #sec-tpm-v2mom-working-group | C08ELA0D39N | 🔒 Private | FY26 V2MOM Process Method working group. |
| #tpm-group-channel | C052MRSSYKF | 🔒 Private | TPM group coordination channel. |
| #tpm-risk-governance | C09FVN9K1QT | 🔒 Private | Risk governance alignment workstream. |
| #tech-prod-tpms | C01LSDCRXGR | 🌐 Public | TPMs in Product COO org. Covers Product, Infra, Security TPMs. |
| #tpm-learning-community | C07EG411R8X | 🌐 Public | TPM best practices, tips, training, AI/Slack productivity. |
| #tpm-allorg | C072HBH5LMB | 🌐 Public | Full combined TPM team. Announcements and virtual water cooler. |

**Total channels:** 9 (9 resolved) — DMs are excluded

---

## Section: `Section8`

API security, OAuth/token containment, inline detection, auto-containment, and related incident channels.

| Channel | ID | Type | Notes |
|---|---|---|---|
| #incident-sev0-response-001 | C0A7FBE4LJF | 🔒 Private | CSOC Bot-created incident channel. Sev0. TLP: RED. ACP. |
| #auto-containment-feature-vpn | C0AL2NKSX0E | 🔒 Private | Auto-containment feature channel for VPN/global case. |
| #case-specific-escalation | C0A8H8FS5S6 | 🔒 Private | Case-specific escalation channel. |
| #sec-advisory-collab | C08H69X1KQE | 🌐 Public | Security advisory for known threat actor targeting. Workflow Q&A. |
| #detection-driven-containment | C0A6W6BJEHM | 🌐 Public | Detection-driven containment workstream. |
| #edge-blocking-sev0-response | C0A1N0US5RR | 🔒 Private | Sev0 response — edge IP blocking. |
| #inline-detections | C06KA4ENG3S | 🔒 Private | Inline detections epic. Also in `security_channels_brief_cron.sh`. |
| #oauth-token-containment-guest-portal | C09UKPWCK9D | 🔒 Private | OAuth token containment — guest portal and integration users. |
| #oauth-token-theft-containment | C0ARK7FHBV3 | 🌐 Public | OAuth token theft containment workstream. |
| #prodsec-seceng-collab | C09L8AFBSJG | 🔒 Private | ProdSec / SecEng collaboration. |
| #project-guardian-h0-tpm-collab | C0A81GNTMNX | 🔒 Private | Guardian H0 TPM collab. Also in `security_channels_brief_cron.sh`. |
| #project-guardian-h0-tip-enforcements | C0AR5N29STB | 🔒 Private | Guardian H0 TIP (Trusted IP) enforcement. |
| #project-tripwire-api-sec | C09KFLUBLF2 | 🔒 Private | Primary Tripwire engineering channel. Also in `tripwire_canvas_cron.sh`, `security_channels_brief_cron.sh`. |
| #project-tripwire-comms-security | C09P41AFVU0 | 🔒 Private | Tripwire comms & security enhancement. Also in `tripwire_canvas_cron.sh`, `security_channels_brief_cron.sh`. |
| #redoubt-leadership-acp | C09VCJL27FW | 🔒 Private | Redoubt leadership / ACP. |
| #redoubt-team-acp | C09V615DEBF | 🔒 Private | Redoubt team-level ACP. |
| #sec-ml-datascience-sync | C020TS6C36H | 🔒 Private | ML/data science sync. Also in `security_channels_brief_cron.sh`. |
| #ext-tor-blocking-collab | ⚠️ UNKNOWN | 🔗 Shared/External | External badge — shared channel, not indexed by Slack API. |
| #sec-log-onboarding-api-logs | C09QFU1QYGN | 🔒 Private | Security log onboarding for API logs. Auto-created by Security-Log Onboarding bot. |
| #temp-api-oauth-user-type | C0A0Z19T64T | 🔒 Private | API/OAuth user type scoped work. |
| #urgent-blocker-guest-user-oauth-containment | C09R30CS07R | 🔒 Private | Urgent blocker — guest user disposition in OAuth token containment. |

**Total channels:** 21 (20 resolved, 1 unresolved — external shared channel not resolvable via API)

---

## Cross-Section Overlap (Existing Cron Coverage)

Channels already covered by active cron scripts — avoid duplicating in new briefs unless cross-section context is desired:

| Channel | ID | Covered By |
|---|---|---|
| #edv-command-center | C0B0LGJ7WE9 | `edv_canvas_cron.sh` |
| #edv-comms-risk-treatment | C0AEBQM3FG8 | `edv_canvas_cron.sh` |
| #email-domain-verification-dev-support | C0AFTGACHRV | `edv_canvas_cron.sh` |
| #guardian-h0-comms-enablement | C0AJW5GLGM9 | `edv_canvas_cron.sh` |
| #sec-risk-treatment-email-abuse | C09RJKBCT0A | `edv_canvas_cron.sh` |
| #temp-domain-verification-mandate | C0AEB4GH37H | `edv_canvas_cron.sh` |
| #temp-edv-lap-execution | C0AKNC0440N | `edv_canvas_cron.sh` |
| #inline-detections | C06KA4ENG3S | `security_channels_brief_cron.sh` |
| #project-guardian-h0-tpm-collab | C0A81GNTMNX | `security_channels_brief_cron.sh` |
| #project-tripwire-api-sec | C09KFLUBLF2 | `tripwire_canvas_cron.sh`, `security_channels_brief_cron.sh` |
| #project-tripwire-comms-security | C09P41AFVU0 | `tripwire_canvas_cron.sh`, `security_channels_brief_cron.sh` |
| #sec-ml-datascience-sync | C020TS6C36H | `security_channels_brief_cron.sh` |
| #sec-ip-reputation-integration | C07N5P6JVQ9 | `security_channels_brief_cron.sh` |

---

## Adding a New Section

Provide a screenshot of the section and say:
> "Add the `_SECTIONNAME` section to `slack_sections.md`"

Claude will resolve all channel IDs via `slack_search_channels` and append the table. DMs and group DMs are always excluded.

**Notes on unresolvable channels:**
- 🔗 **Shared/External** ("External" badge) — these are cross-workspace shared channels. The Slack API cannot find them by search. Retrieve the channel ID manually from the channel URL in Slack.
- Bold entries in the sidebar = unread messages (not a different channel type).
