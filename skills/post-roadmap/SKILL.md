---
name: post-roadmap
description: >
  Write a post-roadmap-meeting follow-up email for a Hallow Parish Success lead, then create
  action items as tasks on the affiliated HubSpot deal. Trigger this skill whenever a Parish
  Success lead says "write roadmap email", "roadmap follow-up", "send the roadmap recap",
  "follow up after roadmap", pastes an Attention call URL from a roadmap meeting, or says
  anything that sounds like they want a post-roadmap-meeting email drafted. Also trigger when
  they say "write a follow-up for [parish name]" after a roadmap meeting context is clear.
  Always use this skill — do not improvise a one-off email draft when this context applies.
---

# Post-Roadmap Meeting Skill

## Required MCPs
- **Attention** (`get_call_details`, `search_calls`, `ask_attention`) — if unavailable, stop immediately and return: "This skill requires the Attention MCP. Please confirm it is connected and try again."
- **HubSpot** (create note, create/patch task, patch deal properties)
- **Google Calendar** (search events for attendee list)

## Required companion skills
- `partner-drive` — resource catalogue with Drive links (used in Step 2b)
- `ps-pipeline` — HubSpot standards, property names, and pipeline IDs (used in Steps 5–6)

**Identity:** This skill is caller-agnostic. The acting Parish Success lead's `hubspot_owner_id` and email address must come from their personal identity skill. Load it before Step 5. Do not hardcode any owner ID or sender email.

---

This skill produces a follow-up email after a Hallow Parish Roadmap Meeting, then logs CRM
records and tasks on the affiliated HubSpot deal.

---

## Step 0 — Require the HubSpot Deal Link (BLOCKING)

Before doing anything else, check whether Bryce has provided a HubSpot deal URL in this
conversation. A valid deal URL looks like:
`https://app.hubspot.com/contacts/4887358/record/0-3/{DEAL_ID}`

**If no deal URL is present:** Stop immediately and ask Bryce for it:

> "Before I pull the call and draft the email, I need the HubSpot deal link for this parish.
> Can you paste it here?"

Do NOT proceed to any subsequent step until a valid deal URL is provided. Extract the
`DEAL_ID` from the URL and store it. All notes, tasks, and CRM records in this session
will be associated with this deal — no company/deal lookup is needed later.

---

## Step 1 — Get the Call

**If Bryce pastes an Attention URL:**
Extract the call ID from the URL (e.g. `https://app.attention.tech/conversations/all-calls/{CALL_ID}`)
and call `Attention:get_call_details` with `detailed_transcript: false`.

**If Bryce gives a parish name (no URL):**
Call `Attention:search_calls` with `title: "[parish name]"` to find the most recent roadmap
meeting for that parish, then call `Attention:get_call_details` on the result.

**CRITICAL — Extract and store the call date immediately from the `get_call_details` response.**
Parse the call's timestamp or date field and store it as `CALL_DATE` (ISO format: YYYY-MM-DD).
Verify the year is correct — it must match the actual year the call took place (read the full
timestamp carefully, not just the month/day). Every task due date in Step 6 is calculated from
this `CALL_DATE`. Do NOT use today's date, and do NOT rely on the date returned by `ask_attention`
in Step 2 — the authoritative source for the call date is always `get_call_details`.

**Pull the Google Calendar event to get attendee emails.**
Search Google Calendar for an event on `CALL_DATE` whose title contains the parish name (or
"roadmap"). Use the attendees list from that event as the authoritative source for email addresses:

- **To:** attendees whose email does NOT end in `@hallow.app`
- **CC:** attendees whose email ends in `@hallow.app`, excluding the acting lead's own email (they are the sender)

If no matching calendar event is found, flag the missing recipients explicitly in the draft
output rather than guessing email addresses.

---

## Step 2 — Analyze the Call

Call `Attention:ask_attention` with the call ID and this prompt:

```
This is a Hallow Parish Roadmap Meeting. Extract:
1. Parish name, call date, and all participant names, roles, and email addresses
2. Current Hallow adoption status at this parish
3. A straightforward summary of what was covered: main topics discussed, decisions made,
   and next steps agreed on
4. Key decisions made during the call — anything both sides agreed on, committed to, or
   ruled out. Be specific about what was decided and why.
5. Verbatim quotes and key phrases used by participants — especially memorable language,
   repeated themes, or phrases that reveal how the parish thinks about their goals. Pull
   exact wording only. If the transcript does not contain clear verbatim quotes, return an
   empty list for this item rather than paraphrasing or inferring.
6. All commitments and next steps mentioned (with dates if given)
7. Parish goals and any pain points surfaced
8. Tone and sentiment of the call
9. Partnership features scheduled or discussed — for EACH of the following, extract the
   specific details if mentioned. Be precise about dates and names; do not infer or guess
   if something was not explicitly stated.

   a. PRAYER CHALLENGES: For each prayer challenge discussed or scheduled, extract:
      - Challenge name (exact title as stated on the call)
      - Scheduled start date (if given — even approximate, e.g. "first Sunday of Advent")
      - Any notes about the challenge (promotional plans, expected participation, etc.)
      If multiple prayer challenges were scheduled, list all of them in order.

   b. PARISH EVENT / PARISH MISSION: If a parish event or mission was scheduled, extract:
      - Event title (e.g. "Parish Mission", "Launch Event", "Advent Mission")
      - Event start date and end date (if given)
      - Event type (e.g. "Launch", "Mission", "Retreat")
      - Any relevant notes (speaker, format, what Hallow is providing, etc.)

   c. GIVING CAMPAIGN: If a giving campaign was discussed or scheduled, extract:
      - Scheduled start date (campaign 1 or campaign 2 if distinguishable)
      - Any notes about the campaign structure, match amount, goals, etc.

   d. MARKETING BOOST: If a Marketing Boost was discussed or scheduled, extract:
      - Scheduled date
      - Target zip codes (if mentioned)
      - Any notes about targeting or goals

   e. PASTOR RECORDING: If a pastor audio recording was discussed, extract:
      - Expected date the recording will be posted
      - Any notes about the recording topic, who is recording, etc.

10. Resource and file needs — based on what was discussed, what documents, guides, or
    resources should be sent to this parish? Be specific (e.g., "they need the bulletin
    template to promote the prayer challenge", "ready to select prayer challenges",
    "priest hasn't recorded his intro yet"). Don't suggest resources generically —
    only flag ones tied to concrete topics from this call.
11. Parish action item due dates — for each action item assigned to the parish, was a
    specific deadline mentioned? If so, record it. If not, note that a standard timeline
    will be applied.

**Important:** For any field not explicitly mentioned on the call, return `null`. Do not infer,
estimate, or guess values — especially for dates, feature names, and contact details.
```

---

## Step 2b — Select Resources for This Email

Read the `partner-drive` skill. Use its **Keyword Trigger Index** for fast lookup —
scan the topics discussed on this call and match them to file numbers. Then pull the full
catalogue entries for only those matched files.

**Selection rule:** Choose up to 3 resources. Prioritize files tied to the most concrete
next steps from this call (e.g. if they committed to scheduling a prayer challenge, #12 is
a stronger pick than a general onboarding doc). If more than 3 files match, pick the 3 most
actionable given where this parish is in the partnership.

For each selected resource, record:
- File number, name, and Drive link (from the catalogue)
- Suggested anchor text
- One sentence on why it's relevant to this call

These will become the Resources section in the email (Step 3) and may also be embedded in
Bryce's action items where applicable.

If no topics from the call match any index entry, omit the Resources section entirely rather
than sending generic files. Never invent or guess URLs — every link must come from the catalogue.

---

## Step 3 — Draft the Email Body

**Email voice:** Warm, direct, and plain. No em-dashes. No corporate filler. No storytelling. Write like a trusted colleague giving a straightforward recap — clear sentences, no embellishment. The sender is the acting Parish Success lead; write in first person from their perspective.

**Recipients:** Use the To/CC split resolved in Step 1 from the Google Calendar event attendee list.

**Subject line format:** `Following Up — [Parish Name] Roadmap Meeting`

Draft ONLY the body of the email — do NOT write an opening or closing. Bryce will write those
himself. Present the draft with clear placeholder labels where his opening and closing go.

**Structure to output:**

```
[YOUR OPENING HERE]

Meeting Summary:
A short, plain recap of what was covered on the call — 3 to 5 sentences. Just say what was
discussed, what was decided, and what the next steps are. No storytelling, no bolded phrases,
no embellishment. Write it the way you'd recap a meeting to a colleague.

Action Items for [Parish Name]:
1. [Action item] — Due: [due date]
2. [Action item] — Due: [due date]
3. [Action item] — Due: [due date]

Action Items for Bryce:
1. ...
2. ...
3. ...

Resources:
Based on what we discussed, here are a few things that may be helpful:
- [Resource Name](URL) — one sentence on why it's relevant
- [Resource Name](URL) — one sentence on why it's relevant
- [Resource Name](URL) — one sentence on why it's relevant

[YOUR CLOSING HERE]
```

The Resources section is populated from Step 2b. Include only the files selected there — up to 3,
each as a linked resource with a brief, specific note on why it applies to this parish right now.
Omit the section entirely if Step 2b found no matches.

### Action Item Guidelines

List only action items that were explicitly discussed or committed to on the call. Do not
invent items to reach a target number. Aim for up to 3 per side, but if only 1 or 2 were
clearly established, list only those.

**For [Parish Name]:** (1–3 items)
Parish-facing tasks — what they need to do before the next touchpoint. Specific and achievable.
Tied to something discussed on the call. These also go into the HubSpot note (Step 5).

Each parish action item must include a due date formatted as `Due: [Month Day]` appended
inline. Assign due dates as follows:
- If a specific deadline was mentioned on the call for this item, use that date.
- If no deadline was mentioned, assign **CALL_DATE + 14 days** as the default.
Never omit due dates from parish action items — they set clear expectations and create
accountability.

**For Bryce:** (1–3 items)
Bryce's commitments — resources to send, invites to schedule, follow-ups to make.
These become HubSpot tasks (Step 6). Create only as many HubSpot tasks as there are real
action items — do not pad to reach 3.

For any action item that involves sending a resource, append the Drive link inline using
the catalogue's suggested anchor text. Format it as a natural sentence, e.g.:
"Send over the [Bulletin Announcement Template](URL) for their upcoming campaign."
Never fabricate URLs — only use links resolved in Step 2b.

---

## Step 4 — HubSpot Association

The `DEAL_ID` was already extracted from the URL provided in Step 0. Use it directly for
all CRM record creation — no company or deal lookup is required.

If for any reason the `DEAL_ID` cannot be parsed from the URL, flag it explicitly and
ask Bryce to confirm the deal ID before proceeding.

---

## Step 5 — Create HubSpot Note (Parish Action Items)

Log a rich-text HTML note on the **deal** documenting the action items assigned to the parish
during the roadmap meeting, including their due dates. This is a reference record of what
the parish is responsible for and when.

**Note format (HTML):**

```html
<h3>Roadmap Meeting — Parish Action Items</h3>
<p><strong>Meeting date:</strong> [date from call]</p>
<p>The following action items were assigned to [Parish Name] during the roadmap meeting:</p>
<ol>
  <li>[Parish action item 1] — <strong>Due: [due date]</strong></li>
  <li>[Parish action item 2] — <strong>Due: [due date]</strong></li>
  <li>[Parish action item 3] — <strong>Due: [due date]</strong></li>
</ol>
<p>A follow-up check-in task has been created for 2 weeks post-meeting.</p>
```

Associate the note with: **deal** only (deal ID from Step 0).

---

## Step 5b — Update Scheduled Partnership Features on the Deal

This step records any partnership features that were scheduled or confirmed on the call directly
onto the HubSpot deal record. Run immediately after Step 5, before creating tasks.

**First, fetch the current deal properties** for the deal ID from Step 0. Pull all prayer
challenge fields (slots 1–8), giving campaign fields, marketing boost fields, pastor recording
fields, and parish event fields. You need to see what is already filled in before writing
anything — this prevents overwriting existing records.

Work through each feature type below. For each one: if the call produced data for it (from
Step 2, item 9), update the deal. If the call produced nothing for a given feature, skip it
entirely — do not blank out or overwrite existing values.

### Prayer Challenges

Prayer challenges are stored in 8 numbered slots on the deal. Each slot has three properties:
name, date, and notes.

**To place a prayer challenge from the call:**
1. Read through slots 1–8 in order. Find the **first slot where the name field is blank** —
   that is the next available slot.
2. Write to that slot:
   - Name → the challenge title from the call
   - Date → the scheduled start date (ISO format YYYY-MM-DD; if only an approximate date
     was given, use your best estimate and note the uncertainty in the notes field)
   - Notes → any context about this challenge (promotional plans, participation goals, etc.)
3. If multiple prayer challenges were scheduled on the call, fill consecutive available slots
   in order.
4. **If all 8 slots are already filled:** Do NOT overwrite anything. Instead, append an
   overflow section to the HubSpot note from Step 5:
   ```html
   <p><strong>Prayer Challenge Overflow:</strong> All 8 prayer challenge slots are full.
   The following challenge(s) were discussed but could not be recorded:
   [challenge name] — [start date] — [notes]. Manual review needed.</p>
   ```

**Property names by slot:**
- Slot 1: `n1st_prayer_challenge_name` / `n1st_prayer_challenge_date` / `n1st_prayer_challenge_notes`
- Slot 2: `n2nd_prayer_challenge_name` / `n2nd_prayer_challenge_date` / `n2nd_prayer_challenge_notes`
- Slot 3: `n3rd_prayer_challenge_name` / `n3rd_prayer_challenge_date` / `n3rd_prayer_challenge_notes`
- Slots 4–8: follow the same pattern with `n4th_` / `n5th_` / `n6th_` / `n7th_` / `n8th_`

### Parish Event / Parish Mission

If a parish event or mission was scheduled on the call, update:
- `mission_title` → event title
- `mission_start_date` → event start date (ISO format, Central Time)
- `mission_end_date` → event end date (ISO format, Central Time) — leave blank if not given
- `parish_event_type` → event type (e.g., "Launch (Growth)", "Mission", "Retreat")
- `parish_event_notes` → any notes about the event

Only write these if `mission_title` is currently blank on the deal. If a `mission_title`
already exists and a new event was discussed, append a note to the Step 5 HubSpot note
rather than overwriting — Bryce may need to decide whether this is a new event or an update.

### Giving Campaign

If a giving campaign start date was discussed:
- If `giving_campaign_1_start_date` is blank → write to it
- If already set → write to `giving_campaign_2_start_date`
- If both are already set → append to `giving_campaign_notes` instead of overwriting
- Always append any new notes from the call to `giving_campaign_notes` (do not replace)

### Marketing Boost

If a marketing boost was discussed:
- If `marketing_boost_date` is blank → write to it
- If already set → write to `marketing_boost_2_date`
- Write zip codes (if mentioned) to `marketing_boost_zip_codes`
- Append any notes to `marketing_boost_notes`

### Pastor Recording

If a pastor recording was discussed:
- If `pastor_recording_1_date` is blank → write the expected posting date to it
- If already set → write to `pastor_recording_2`
- Append any notes to `pastor_recording_notes`

### Feature Tasks — Create HubSpot Tasks for Scheduled Features

Immediately after writing the deal properties above, create a HubSpot task for each feature
that was actually scheduled on this call. These tasks are conditional — only create them for
features that produced a concrete date from Step 2, item 9. Skip any feature where no date
was established.

These tasks are in addition to Tasks 1–5 in Step 6. Create them without asking for confirmation
(`confirmationStatus: CONFIRMATION_WAIVED_FOR_SESSION`). Associate all of them with the **deal**
only (deal ID from Step 0). Use the acting lead's `hubspot_owner_id` (from their identity skill) on all.

For each task, set `hs_timestamp` to the feature's scheduled date **minus 14 days** (two weeks
out gives Bryce prep time). If the scheduled date is less than 14 days away or has already
passed, use the scheduled date itself. Convert all due dates to Unix epoch milliseconds.

#### Giving Campaign Task
Create only if a giving campaign start date was established on the call.

| Property | Value |
|---|---|
| `hs_task_subject` | `Giving Campaign prep — [Parish Name] (starts [campaign start date])` |
| `hs_task_type` | `TODO` |
| `hs_timestamp` | Campaign start date − 14 days |
| `hs_task_status` | `NOT_STARTED` |
| `hs_task_priority` | `HIGH` |
| `hs_task_body` | `The giving campaign at [Parish Name] starts [campaign start date]. Confirm promotional materials are in place, the parish has what they need, and any Hallow-side setup is complete.` |

#### Marketing Boost Task
Create only if a marketing boost date was established on the call.

| Property | Value |
|---|---|
| `hs_task_subject` | `Marketing Boost prep — [Parish Name] ([boost date])` |
| `hs_task_type` | `TODO` |
| `hs_timestamp` | Boost date − 14 days |
| `hs_task_status` | `NOT_STARTED` |
| `hs_task_priority` | `HIGH` |
| `hs_task_body` | `Marketing Boost scheduled for [Parish Name] on [boost date]. Confirm zip codes, targeting, and any parish-side promotion are lined up ahead of the launch.` |

#### Parish Event Task
Create only if a parish event or mission was scheduled on the call.

| Property | Value |
|---|---|
| `hs_task_subject` | `[Event title] prep — [Parish Name] (starts [event start date])` |
| `hs_task_type` | `TODO` |
| `hs_timestamp` | Event start date − 14 days |
| `hs_task_status` | `NOT_STARTED` |
| `hs_task_priority` | `HIGH` |
| `hs_task_body` | `[Event title] at [Parish Name] starts [event start date]. Confirm Hallow's role is fully set up, any resources or recordings are ready, and the parish has everything they need going in.` |

### After updating all features

In Step 7b, report a clean summary of what was written (or skipped). For example:
"Prayer Challenge Slot 3: Advent with Augustine — Dec 1, 2026" or "Giving Campaign #1
already set — appended call notes only." Also list any feature tasks created here, with their
due dates, so Bryce has a complete picture without opening HubSpot.

---

## Step 5c — Create Pinned Deal Note (Meeting Summary)

Create a pinned summary note on the deal that replaces whatever is currently pinned at the top
of the activity feed. HubSpot only allows one pinned note per record, so setting
`hs_note_is_pinned: true` on the new note will automatically displace any existing one.

This note is the at-a-glance reference for anyone (including future-Bryce) who opens this deal
and wants to immediately understand where things stand.

**Note format (HTML):**

```html
<h3>Roadmap Meeting Summary</h3>
<p><strong>Meeting date:</strong> [CALL_DATE formatted as Month Day, Year]</p>

<h4>Conversation Summary</h4>
<p>[3–5 sentence plain-language recap of the full conversation: what was discussed, what was
decided, and the general tone and direction of the partnership. Write it like you're briefing
a colleague who wasn't on the call — factual, specific, no fluff.]</p>

<h4>Action Items for Bryce</h4>
<ol>
  <li>[Bryce action item 1] — <strong>Due: [due date]</strong></li>
  <li>[Bryce action item 2] — <strong>Due: [due date]</strong></li>
  <li>[Bryce action item 3] — <strong>Due: [due date]</strong></li>
</ol>

<h4>Action Items for [Parish Name]</h4>
<ol>
  <li>[Parish action item 1] — <strong>Due: [due date]</strong></li>
  <li>[Parish action item 2] — <strong>Due: [due date]</strong></li>
  <li>[Parish action item 3] — <strong>Due: [due date]</strong></li>
</ol>

<h4>Major Next Step</h4>
<p><strong>[One sentence: the single most important thing that needs to happen next — from
either Bryce or the parish — to move this partnership forward. Be concrete and specific.]</strong></p>

<p><strong>Call Recording:</strong> <a href="[Attention call URL]">View Call on Attention</a></p>
```

Only include as many action items as actually exist — do not pad with placeholder rows.
The "Major Next Step" should cut through the full list and name the one thing that, if it
happens, makes everything else easier. It might be Bryce's responsibility or the parish's —
pick whatever is genuinely most critical.

The Attention call URL is the full conversation URL from Step 1:
`https://app.attention.tech/conversations/all-calls/{CALL_ID}`. Use it as the href for the
"View Call on Attention" link — this gives anyone opening the deal a direct path back to the
recording without having to search.

Associate the note with: **deal** only (deal ID from Step 0). Set `hs_note_is_pinned: true`.

---

## Step 6 — Create HubSpot Tasks

Follow **ps-pipeline** as the authoritative reference for all task creation — required
fields, property names, association lookup order, and standards all live there. What's specified
below is roadmap-email-specific guidance layered on top.

Create all 5 tasks without asking for additional confirmation
(`confirmationStatus: CONFIRMATION_WAIVED_FOR_SESSION`).

**DATE RULE — applies to every task below:**
All `hs_timestamp` values must be computed from `CALL_DATE` (extracted in Step 1 from
`get_call_details`). Convert `CALL_DATE` to a Unix epoch timestamp in milliseconds for the API.
Tasks 1–4 must fall between `CALL_DATE` and `CALL_DATE + 14 days`. Task 5 falls at `CALL_DATE + 21 days`.

**Timestamp verification (blocking — do not submit any task until this is complete):**
Before submitting, output each computed timestamp alongside its human-readable date in this
format:

```
Task 1: 1747180800000 → 2026-05-14 ✓ (matches CALL_DATE year, within 14 days)
Task 2: 1747180800000 → 2026-05-14 ✓
Task 3: 1747180800000 → 2026-05-14 ✓
Task 4: 1747785600000 → 2026-05-21 ✓ (CALL_DATE + 14 days)
Task 5: 1748390400000 → 2026-06-04 ✓ (CALL_DATE + 21 days)
```

Confirm all five before proceeding. If any timestamp resolves to a date that:
- Predates today's date → it is wrong, recompute from scratch
- Falls in a different year than `CALL_DATE` → it is wrong, recompute from scratch
- Task 1–4 falls more than 14 days after `CALL_DATE` → it is wrong, recompute from scratch
- Task 5 falls more than 21 days after `CALL_DATE` → it is wrong, recompute from scratch

Do not skip this output step or proceed past it until all timestamps are verified.

### Tasks 1–3: Bryce's Action Items from the Call

| Property | Value |
|---|---|
| `hs_task_subject` | Clear action verb + parish name (e.g. "Send Partner Manual to St. Michael") |
| `hs_task_type` | `EMAIL`, `CALL`, or `TODO` — match to the action |
| `hs_timestamp` | If a specific due date was mentioned on the call for this task, use that date. Otherwise: **CALL_DATE + 7 days** (one week after the call). |
| `hs_task_status` | `NOT_STARTED` |
| `hubspot_owner_id` | Acting lead's owner ID (from their identity skill) |
| `hs_task_priority` | `HIGH` for meeting-related items, `MEDIUM` for resource sends |
| `hs_task_body` | 1–2 sentences: what this is, why it matters, what "done" looks like. If a Drive resource was matched for this task, include the URL in the body so it's available in HubSpot without reopening the email. |

### Task 4: Parish Follow-Up Check-In

| Property | Value |
|---|---|
| `hs_task_subject` | `Follow up with [Parish Name] on roadmap action items` |
| `hs_task_type` | `EMAIL` |
| `hs_timestamp` | **CALL_DATE + 14 days** (two weeks after the call) |
| `hs_task_status` | `NOT_STARTED` |
| `hubspot_owner_id` | Acting lead's owner ID (from their identity skill) |
| `hs_task_priority` | `HIGH` |
| `hs_task_body` | A ready-to-send check-in email drafted in Bryce's voice (see below). |

The `hs_task_body` for this task must contain a complete, ready-to-send email — not a
description of what to write. When Bryce opens this task in HubSpot two weeks later, he should
be able to copy the body and send it with minimal edits.

Draft the email using the same voice as Step 3: warm, direct, no em-dashes. The format is fixed — use it exactly. Pull the parish POC emails from Step 1's attendee list for the To: field.

**Email format for the task body:**

```
To: [parish POC email addresses from Step 1, comma-separated]
CC: [Hallow attendees from Step 1 excluding the acting lead's email, or omit if none]
Subject: Checking In — [Parish Name] Action Items

Hey Team,

I just wanted to touch base on your three most recent action items.

1. [Parish action item 1] — due [due date],
2. [Parish action item 2] — due [due date],
3. [Parish action item 3] — due [due date].

Blessings,
```

Keep it exactly this short. No preamble, no explanation — just the three items with due dates.
Only include action items assigned to the parish (not Bryce's own items). Use lowercase "due"
inline (matching the example above), not "Due:" on a separate line.

### Task 5: ✨Send Delightful Email✨

| Property | Value |
|---|---|
| `hs_task_subject` | `✨Send Delightful Email✨ — [Parish Name]` |
| `hs_task_type` | `EMAIL` |
| `hs_timestamp` | **CALL_DATE + 21 days** (three weeks after the call) |
| `hs_task_status` | `NOT_STARTED` |
| `hubspot_owner_id` | Acting lead's owner ID (from their identity skill) |
| `hs_task_priority` | `MEDIUM` |
| `hs_task_body` | `Three weeks post-roadmap — time to send a warm, delightful check-in to [Parish Name]. Celebrate any wins, acknowledge progress on their action items, and keep the momentum going. This is a relationship touch, not a task follow-up.` |

Associate all tasks with: **deal** only (deal ID from Step 0).

---

## Step 7 — Present Results

### 7a — Display the Email Draft in Chat

Output the complete email draft directly in chat. Do NOT create an artifact. Format it
clearly so Bryce can copy and paste it into Gmail.

**Format:**

```
Subject: Following Up — [Parish Name] Roadmap Meeting

To: [recipient emails]
CC: [cc emails, or omit if none]

---

[YOUR OPENING HERE]

[Full email body — Meeting Summary, Action Items for Parish (each with Due: date),
 Action Items for Bryce, Resources section if applicable]

[YOUR CLOSING HERE]
```

### 7b — Chat Confirmation

After the email draft, post a brief summary:

1. **HubSpot confirmation** — brief list of the note + tasks created, with deal link:
   `https://app.hubspot.com/contacts/4887358/record/0-3/{DEAL_ID}`
2. **Features logged** — a short summary of what was written to the deal in Step 5b:
   - Each prayer challenge slot filled (slot number, challenge name, start date)
   - Each partnership feature property updated (giving campaign, marketing boost, pastor recording, parish event)
   - Any overflow warnings or skipped fields (with reason)
   Omit this section entirely if no features were scheduled on this call.