---
name: sort-prayer-challenges
description: >
  Sort a HubSpot parish deal's prayer challenges into chronological order by launch date.
  Use this skill whenever Bryce pastes a HubSpot deal URL and says anything like "sort the
  prayer challenges", "reorder the challenges", "fix the challenge order", "put these in
  date order", or when prayer challenges on a deal appear to be in the wrong sequence.
  Also trigger when Bryce says "the challenges are out of order" or asks to "clean up the
  prayer challenge slots" on a deal.
---
 
# Sort Prayer Challenges by Launch Date
 
This skill reorders a deal's prayer challenge slots (1–8) so they're in ascending launch date order. Each slot has three fields that travel together: name, date, and notes.
 
## HubSpot Property Names
 
These are the exact API property names — not what you see in the UI.
 
| Slot | Name property | Date property | Notes property |
|------|--------------|---------------|----------------|
| 1st | `n1st_prayer_challenge_name` | `n1st_challenge__launch_date` | `n1st_prayer_challenge_notes` |
| 2nd | `n2nd_prayer_challenge_name` | `n2nd_challenge__launch_date` | `n2nd_prayer_challenge_notes` |
| 3rd | `n3rd_prayer_challenge_name` | `n3rd_challenge__launch_date` | `n3rd_prayer_challenge_notes` |
| 4th | `n4th_prayer_challenge_name` | `n4th_challenge__launch_date` | `n4th_prayer_challenge_notes` |
| 5th | `n5th_prayer_challenge_name` | `n5th_challenge__launch_date` | `n5th_prayer_challenge_notes` |
| 6th | `n6th_prayer_challenge_name` | `n6th_challenge__launch_date` | `n6th_prayer_challenge_notes` |
| 7th | `n7th_prayer_challenge_name` | `n7th_challenge__launch_date` | `n7th_prayer_challenge_notes` |
| 8th | `n8th_prayer_challenge_name` | `n8th_prayer_challenge__date` | `n8th_prayer_challenge_notes` |
 
**Note:** The 8th slot's date property is named differently (`n8th_prayer_challenge__date`, not `n8th_challenge__launch_date`). Don't mix these up.
 
## Workflow
 
### 1. Extract the deal ID from the URL
 
The deal ID is the number after `/0-3/` in the HubSpot URL.
Example: `https://app.hubspot.com/contacts/4887358/record/0-3/52037683997` → deal ID `52037683997`
 
### 2. Fetch all challenge slots
 
Call `get_crm_objects` for the deal with all 24 prayer challenge properties (8 slots × 3 fields each). Fetch only populated slots — if a name field is empty, that slot is unused and should be ignored.
 
### 3. Sort by launch date
 
Sort the populated slots by their date property ascending. If two challenges share the same date, preserve their current relative order.
 
### 4. Compare new order to current order
 
Identify which slots changed position. Only slots that actually moved need to be updated.
 
### 5. Show a confirmation table
 
Before writing anything, show Bryce a table of proposed changes. Only include rows for slots that are changing. Format:
 
| Field | Current Value | New Value |
|-------|--------------|-----------|
| 4th Name | Old Name | New Name |
| 4th Date | Old Date | New Date |
| 4th Notes | Old notes... | New notes... |
 
Wait for explicit approval before proceeding.
 
### 6. Update the deal
 
Write all changed slot fields in a single `manage_crm_objects` call. Make sure each slot's name, date, and notes all update together — never update just one field of a slot without the others.
 
### 7. Confirm
 
After the update succeeds, show the final ordered list so Bryce can verify at a glance:
 
| # | Challenge | Launch Date |
|---|-----------|-------------|
| 1 | ... | ... |
...
 
## Edge Cases
 
- **Empty slots in the middle**: If slot 3 is empty but slot 4 has data, treat each slot independently — sort only the populated ones and write them back into slots 1, 2, 3... consecutively. Don't leave gaps.
- **Challenges already in order**: Let Bryce know everything looks correct and no changes are needed.
- **Missing date on a slot**: If a slot has a name but no date, flag it to Bryce before proceeding — don't guess at ordering for dateless slots.
