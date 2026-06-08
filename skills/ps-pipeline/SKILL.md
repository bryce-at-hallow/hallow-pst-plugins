---
name: ps-pipeline
user-invocable: false
description: >-
  HubSpot standards and process reference for the Hallow Parish Success team.
  Covers: Parish Success Pipeline ID and stage IDs, task creation requirements,
  note conventions, deal/company property reference, overdue parish definition,
  and HubSpot MCP capability limits. Source of truth — consult before creating
  or updating any HubSpot record. Personal identity (owner IDs) comes from the
  acting lead's identity skill. Use whenever a Parish Success lead creates or
  updates a task, looks up a deal or company, logs a note, scans a portfolio
  for overdue parishes, or performs any HubSpot operation on a parish record.
---

**Today's date: !`date +%Y-%m-%d`**

# Parish Success Pipeline — HubSpot Standards

This skill defines the shared HubSpot process for the Parish Success team. Personal identity (owner ID, account link format) comes from each lead's own identity skill — supply it at task time.

---

## Parish Success Pipeline

**Pipeline ID:** `109690430`

Always filter parish deals to this pipeline when searching for a parish's active deal.

### Pipeline Stages

| Stage Label | Stage ID |
|---|---|
| New Partner | `196439454` |
| Ops Complete | `1198066858` |
| Onboarding Mtg Scheduled | `196439456` |
| Onboarding Mtg Complete | `196439457` |
| Roadmap Mtg Scheduled | `1048951725` |
| Roadmap Mtg Complete | `1048951726` |
| Event Scheduled | `1048951727` |
| Event Complete | `1048951728` |
| Mid-Year Check-In Scheduled | `1048951729` |
| Mid-Year Check-In Complete | `196439458` |
| Renewal Conversations | `196439578` |
| Renewed | `196439580` |
| Not Renewed | `196439581` |

---

## Task Creation Standard

This is the **authoritative reference for all HubSpot task creation** across the Parish Success team. Any other skill (attention-roadmap-email, hubspot-parish-audit, morning-coffee, etc.) that creates tasks must defer to the standards here. Skill-specific guidance (due date windows, task counts) layers on top of this baseline but never overrides it.

Every task must have ALL of the following fields populated. Never leave any blank. Never create a task without resolving all associations first.

### Required Properties

| Property | Internal Name | Notes |
|---|---|---|
| Title | `hs_task_subject` | Always include parish name. Clear and action-oriented. |
| Task Type | `hs_task_type` | Must be `TODO`, `EMAIL`, or `CALL` — always explicit, never ambiguous |
| Due Date | `hs_timestamp` | Always future-dated from today. Default 3 business days unless context specifies otherwise. Never hardcode a year. |
| Status | `hs_task_status` | Always `NOT_STARTED` on creation |
| Owner | `hubspot_owner_id` | Set to the acting Parish Lead's owner ID — supplied by their personal identity skill |
| Priority | `hs_task_priority` | Always `HIGH`, `MEDIUM`, or `LOW` — never blank |
| Body / Notes | `hs_task_body` | Include: background context, what needs to happen, and why |

> **CRITICAL — Task Due Dates:** Task due dates must always be future-dated from today's actual date. The current date is available in your system context (`currentDate`). **Always read `currentDate` from system context — never use training knowledge to infer the year.** Before setting any `hs_timestamp`, state today's date aloud (e.g. "Today is 2026-06-08"), then compute forward from it. Tasks set in the past will not appear in HubSpot's task queue and will be invisible. Year errors (e.g. setting 2025 when today is 2026) are a known failure mode — verify the year before submitting.

### Required Association

Tasks must be associated with the **Deal** on the Parish Success Pipeline only.

| Association | How to Resolve |
|---|---|
| **Deal** | Search deals filtered to pipeline `109690430` by parish name → get ID → associate |

If the deal cannot be resolved, flag it explicitly rather than silently omitting it.

### Association Lookup Order

Before creating any task, always run this sequence:

1. Search deals where `pipeline = 109690430` by parish name → get `deal_id`
2. Create the task associated with the deal

---

## Note Creation Standard

Note timestamp = today's actual date and time. Never backdate. HubSpot's activity timeline sorts by timestamp — a backdated note will be buried and invisible in the feed.

- Notes go on the **deal** record, not the company
- Use **rich text HTML** for all note bodies logged to HubSpot

> **Rule:** Note date = today. Always.

---

## Deal Properties Reference

When pulling or working with a deal record, always include these properties.

### Identity & Ownership
- `dealname`, `hubspot_owner_id`, `pipeline`, `dealstage`

### Financial
- `amount`, `calculated_tcv`, `calculated_acv`, `sum_paid_rollup`, `line_item_product`, `deal_product`

### Contract
- `start_date`, `end_date`, `contract_status`, `contract_length_months`

### Parish Info
- `primary_company_name`, `primary_company_id_deal`, `state_primary_company`, `primary_company_diocese`

### Billing
- `billing_poc_firstname`, `billing_poc_lastname`, `billing_poc_email`

### Health & Engagement
- `parish_partnership_quality_rating` — Partnership Quality Rating
- `notes_last_contacted` — Last Contacted (auto-set by HubSpot; reflects last logged call, email, or meeting on the deal record)
- `notes_next_activity_date` — Next Activity Date
- `hs_next_step` — Next Step

### Parish Event
- `mission_start_date`, `mission_end_date`, `mission_title`, `parish_event_type`, `parish_event_notes`, `parish_event_url`, `starter_sheet_link`

If these fields are empty, the parish has not yet utilized their Parish Event feature.

### Giving Campaign
- `stripe_account_created_date`, `stripe_account_link`, `stripe_account_id`
- `giving_campaign_1_start_date`, `giving_campaign_2_start_date`
- `giving_campaign_match`, `giving_campaign_notes`, `giving_campaign_webinar_date`

If these fields are empty, the parish has not yet utilized their Giving Campaign feature.

### Marketing Boost
- `marketing_boost_date`, `marketing_boost_2_date`, `marketing_boost_zip_codes`, `marketing_boost_notes`

If these fields are empty, the parish has not yet utilized their Marketing Boost feature.

### Pastor Recordings
- `pastor_recording_1_date`, `pastor_recording_2`, `pastor_recording_notes`

If these fields are empty, the parish has not yet utilized their Pastor Recording feature.

### Prayer Challenges (8 windows — name, date, notes each)
- `n1st_prayer_challenge_name` / `n1st_prayer_challenge_date` / `n1st_prayer_challenge_notes`
- `n2nd_prayer_challenge_name` / `n2nd_prayer_challenge_date` / `n2nd_prayer_challenge_notes`
- `n3rd_prayer_challenge_name` / `n3rd_prayer_challenge_date` / `n3rd_prayer_challenge_notes`
- `n4th_prayer_challenge_name` / `n4th_prayer_challenge_date` / `n4th_prayer_challenge_notes`
- `n5th_prayer_challenge_name` / `n5th_prayer_challenge_date` / `n5th_prayer_challenge_notes`
- `n6th_prayer_challenge_name` / `n6th_prayer_challenge_date` / `n6th_prayer_challenge_notes`
- `n7th_prayer_challenge_name` / `n7th_prayer_challenge_date` / `n7th_prayer_challenge_notes`
- `n8th_prayer_challenge_name` / `n8th_prayer_challenge_date` / `n8th_prayer_challenge_notes`

---

## Company Page Usage

Use the company page — not the deal — as the primary source of truth for last-touch date. Activity logged against a company (emails sent to contacts, calls, notes) does not always roll up to the deal's `notes_last_contacted`.

### Key Last-Touch Properties — Pull from COMPANY, not deal

| Property | Internal Name | What It Captures |
|---|---|---|
| Last Contacted | `notes_last_contacted` | Last logged call, email, or meeting |
| Last Booked Meeting | `hs_last_booked_meeting_date` | Most recent meeting booked |
| Last Sales Activity | `hs_last_sales_activity_timestamp` | Broadest last activity signal |
| Last Updated | `notes_last_updated` | Last note or activity logged |

Use the **most recent value across all four** as the true last touch date.

### "Overdue" Definition

A parish is only considered overdue if ALL of the following are true:

1. Most recent last-touch signal on the company record is 30+ days ago
2. No upcoming meetings scheduled (`MEETING_EVENT` where `hs_meeting_start_time >= today` associated with the company)
3. No `notes_next_activity_date` set on the deal

If any one of these has a future date or upcoming activity, the parish is **not** overdue — there is a plan in place. Never flag a parish as overdue based on deal-level `notes_last_contacted` alone.

### Bulk Portfolio Scan Caveat

Filtering by deal-level `notes_last_contacted` is a useful first pass, but it produces false positives — the company record may show recent activity that didn't roll up to the deal. Always verify at the company level for any parish flagged as overdue before surfacing it. Present bulk results as "candidates to verify," not confirmed overdue parishes.

### Campus Type Property
- Internal name: `parish_type`
- Valid values: `Standard Parish`, `School Only`, `Cluster`, `Newman Center`, `Cluster of Parishes + Schools`, `Multiple Schools`, `Multiple Schools + Parish`

---

## HubSpot MCP Capabilities

### Available (read + write)
Deals, Companies, Contacts, Notes, Tasks, Meetings, Calls, Emails, Tickets, Line Items

### NOT available
- Meeting scheduling links — direct users to app.hubspot.com/meetings/
- Email sequences or workflows
- Marketing emails or lists
- Reporting dashboards
- Quotes, invoices, orders (read-only or not available on account tier)
- Products (permission restriction)

---

## Key Rules

- Always filter owned records using the acting lead's `hubspot_owner_id`
- Never create a task without resolving the deal association first
- Never leave priority blank — always assign HIGH, MEDIUM, or LOW
- Always include a body note on tasks — context is required, not optional
- Notes always go on the deal, not the company
- Rich text HTML for all note bodies logged to HubSpot
- Parish Success Pipeline ID is `109690430` — always use this when searching for active parish deals
- Never set a task due date in the past — tasks in the past are invisible in HubSpot's queue
- Note timestamps must always be today's date — never backdate a note