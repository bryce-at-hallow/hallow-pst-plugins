# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

A Claude Code plugin (`hallow-pst-plugin`) for the Hallow Parish Success team. It contains skills that automate HubSpot workflows, meeting follow-ups, and CRM data management via Claude Code.

Plugin metadata lives in [.claude-plugin/plugin.json](.claude-plugin/plugin.json).

## Skill Structure

Each skill lives in `skills/<skill-name>/SKILL.md`. Skills are markdown files with YAML frontmatter:

```yaml
---
name: skill-name
description: >
  Trigger description — used for auto-invocation matching.
  Be specific about when to trigger vs. not.
user-invocable: false  # omit if user-invocable
---
```

Body is the instruction set the model follows when the skill runs.

## Skills in This Repo

| Skill | Purpose |
|---|---|
| `ps-pipeline` | HubSpot standards reference — pipeline ID, stage IDs, task creation rules, overdue parish definition. **Non-invocable; loaded as context by other skills.** |
| `sort-prayer-challenges` | Reorders prayer challenge slots 1–8 on a HubSpot deal by launch date. |
| `post-roadmap` | Full post-roadmap-meeting workflow: pulls Attention call, drafts follow-up email, creates HubSpot note + tasks, updates deal properties. |
| `hello` | Dev/test skill — greets a named user. |

## Key Domain Context

**Parish Success Pipeline ID:** `109690430` — always filter deals to this pipeline.

**Prayer challenge slots:** 8 numbered slots per deal (1st–8th), each with name/date/notes properties. The 8th slot's date field uses a different naming convention (`n8th_prayer_challenge__date`, not `n8th_challenge__launch_date`).

**Task creation rules** (from `ps-pipeline`):
- All tasks need: subject, type (TODO/EMAIL/CALL), future-dated timestamp, NOT_STARTED status, owner ID, priority (HIGH/MEDIUM/LOW), and body
- Always associate tasks with the **deal**, not the company
- Notes go on the **deal** in rich-text HTML; never backdate them
- Overdue = 30+ days since last company-level touch AND no upcoming meetings AND no `notes_next_activity_date`

**Bryce's HubSpot owner ID:** `83821452`

**HubSpot account:** `4887358` (appears in all deal URLs: `https://app.hubspot.com/contacts/4887358/record/0-3/{DEAL_ID}`)

## Writing Skills

- Trigger descriptions in frontmatter should be explicit about what NOT to trigger on (avoids false positives)
- Steps that write to HubSpot should require deal URL first (blocking gate) — see `post-roadmap` Step 0 as the pattern
- Task due dates: always compute from a real date variable, never hardcode year; verify timestamps before submitting
- Skills that create HubSpot records should reference `ps-pipeline` for the authoritative field list
- Skills referencing other skills by name (e.g. `bryce-email-voice`, `parish-partner-drive`) are dependencies that live outside this repo
