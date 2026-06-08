# hallow-pst-plugin

Claude Code plugin for the Hallow Parish Success team. Automates HubSpot workflows, meeting follow-ups, and CRM data management.

## Installation

```
/plugin install bryce-at-hallow/hallow-pst-plugins
/reload-plugins
```

## Skills

### `post-roadmap`

Runs the full post-roadmap-meeting workflow end to end.

**What it does:**
1. Pulls the call transcript from Attention using the meeting URL
2. Extracts parish info, attendees, features discussed, and action items
3. Drafts a follow-up email in the lead's voice
4. Creates a HubSpot note on the deal (rich-text HTML, never backdated)
5. Creates all follow-up tasks on the deal — immediate action items, a 2-week check-in, a 3-week relationship touch, and feature prep tasks for anything with a scheduled launch date

**Trigger phrases:** "write roadmap email", "roadmap follow-up", "follow up after roadmap", paste an Attention call URL

**Requires:** Attention MCP, HubSpot MCP, Google Calendar MCP, the lead's identity skill, `ps-pipeline`, `partner-drive`, and `bryce-email-voice`

---

### `sort-prayer-challenges`

Reorders the 8 prayer challenge slots on a HubSpot parish deal into ascending launch date order.

Each slot carries three fields together (name, date, notes). The skill reads all 8 slots, sorts by date, then patches them back in the correct sequence. Handles the 8th slot's non-standard date field name (`n8th_prayer_challenge__date`).

**Trigger phrases:** "sort the prayer challenges", "reorder the challenges", "fix the challenge order", "the challenges are out of order"

**Requires:** HubSpot MCP, a deal URL

---

### `partner-drive`

Reference catalogue of all Hallow Parish Partner Drive files. Provides Google Drive links with context on what each file is and when to share it.

Covers: Partner Manual, bulletin/pulpit templates, Herald resources, event guides, subscription docs, prayer challenge list, founder videos.

Used automatically by `post-roadmap` and email skills whenever an action item involves sending a file. Never hallucinate URLs — all links come from this catalogue.

**Trigger phrases:** "send them the manual", "share the herald doc", "what's the link for X", "attach the subscription guide"

---

### `ps-pipeline` *(internal — not user-invocable)*

Source-of-truth reference for HubSpot standards on the Parish Success team. Other skills load this as context before creating or updating any HubSpot record.

Covers:
- Parish Success Pipeline ID (`109690430`) and all stage IDs
- Task creation requirements (required fields, association rules, timestamp rules)
- Note conventions (deal-level, rich-text HTML, never backdated)
- Deal and company property reference
- Overdue parish definition (30+ days no touch, no upcoming meetings, no `notes_next_activity_date`)
- HubSpot MCP capability limits

## Key Context

| Detail | Value |
|---|---|
| Parish Success Pipeline ID | `109690430` |
| HubSpot account | `4887358` |
| Deal URL format | `https://app.hubspot.com/contacts/4887358/record/0-3/{DEAL_ID}` |
| Tasks associate to | Deal (not company) |
| Notes associate to | Deal (not company) |

## Repo Structure

```
skills/
  post-roadmap/           # Post-roadmap meeting workflow
  sort-prayer-challenges/ # Reorder prayer challenge slots
  partner-drive/          # Partner Drive file catalogue
  ps-pipeline/            # HubSpot standards reference (internal)
  hello/                  # Dev/test skill
.claude-plugin/
  plugin.json             # Plugin manifest
```
