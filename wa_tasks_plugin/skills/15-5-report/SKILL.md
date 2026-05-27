---
name: 15-5-report
description: Draft a 15-5 weekly report for the user by pulling this week's activity from connected tools (Fathom, Gmail, Google Drive, Basecamp, Linear, Slack, Teams, Zoom — whichever are connected) and filling in the West Arete 15-5 template. Use this skill whenever the user asks for a "15-5", "fifteen-five", "15/5 report", "weekly 15-5", "my 15-5 for this week", or any phrasing involving the 15-5 ritual. Trigger even when the user says only "do my 15-5" or "write the 15-5" — this is the default reading for those phrases. Do not use this skill for a generic "Friday status" or "weekly readout" — those have their own skills with different templates.
---

# 15-5 report

Draft a 15-5 weekly report for the user. The 15-5 is a West Arete ritual based on Yvon Chouinard's Patagonia practice: a weekly write-up that takes about fifteen minutes to write and five minutes to read. It uses a fixed six-section template and is posted for the user's manager at end of week.

The goal of this skill is to do the heavy data-gathering across connectors so the user can spend their fifteen minutes on the reflective parts (feelings, lessons, kudos) rather than on remembering what they did.

## Required connectors

Check which of these are connected. Use whatever is available; note in the final output any that are missing so the user knows what wasn't covered.

- Fathom — recorded meetings, transcripts, summaries
- Gmail — threads the user sent or starred this week
- Google Drive — documents the user edited
- Basecamp — posts, todos completed, comments
- Linear — issues moved, comments authored
- Slack — messages and threads (if connected)
- Microsoft Teams — messages and threads (if connected)
- Zoom — meeting recordings/summaries (if connected)

If Slack, Teams, or Zoom are not connected, do not stop — proceed with what is available and add a one-line note at the end of the output listing the connectors that were unavailable.

## The template

The output MUST follow this exact structure and ordering. Do not rename, reorder, or add sections. Section headings are plain text (not markdown headers) to match the format the user pastes into Basecamp.

```
Week of YYYY-MM-DD [the Monday of this week]

How did you feel at work this week?

*

What did you accomplish this week?
List your completed activities and accomplishments. What is working well?

*

What are your priorities for next week?
Be specific.

*

What are your biggest challenges right now?
And how might we help?

*

What lessons did you learn? What opportunities do you see for improvement?
These may be lessons for yourself, or insights that might benefit other parts of the company. What questions are you trying to solve? Any ideas for how to improve the company?

*

Any kudos that you'd like to give?
Appreciate teammates by sharing the impact they had. If appropriate, mention how their actions aligned with our company values.

*
```

## Workflow

### 1. Determine the week

Compute the Monday of the current week and use it as the "Week of" date. The reporting window is Monday 00:00 through today (or through Friday if today is later). Resolve to ISO dates and pass those literal dates to connector tools — never pass relative phrases like "this week".

If the user explicitly names a different week, honor that and recompute the Monday.

### 2. Gather activity in parallel

Make these calls as a single batch when the tool harness allows it:

**Fathom** — `list_meetings` or `search_meetings` bounded to the window. For each meeting, prefer `get_meeting_summary` over `get_meeting_transcript`. Capture title, date, attendees, recording URL, decisions, action items assigned to the user, and outcomes.

**Gmail** — `search_threads` with date-bounded queries. Run at least two queries: (a) threads where the user sent a substantive reply (label `SENT`, date-bounded), and (b) starred threads in the window. Skip routine notifications and calendar invites.

**Google Drive** — `list_recent_files` filtered to items the user modified in the window. Capture title, link, and a one-line "what changed" note if the doc is short enough to read.

**Basecamp** — for each project the user participates in, use `get_events` scoped to the window, or `global_search` if event filtering is unclear. Capture posts the user authored, todos the user completed, and substantive comments the user left.

**Linear** — `list_issues` filtered by assignee = user and `updatedAt` in the window. Capture issues that moved to In Review or Done, and comments the user authored. Resolve the user via `get_user` if needed.

**Slack / Teams / Zoom** — if connected, pull activity by the user in the window. If not connected, skip.

### 3. Synthesize into the template

Fill each section as follows.

**Week of** — the Monday date in `YYYY-MM-DD` format.

**How did you feel at work this week?** — leave the bullet as `*` (empty) followed by a single inline placeholder: `[your reflection — only you can write this]`. Do not invent feelings. This is the user's job.

**What did you accomplish this week?** — this is where most of the connector data lands. Group items by theme, not by source tool. One bullet per accomplishment. Each bullet names the work in plain language, adds one sentence of context if the title is not self-explanatory, and links to the source as `[Title](URL)`. Merge items that reference the same underlying work (e.g., a Linear ticket and the Drive doc it produced collapse to one bullet). If a meeting produced a decision, lead with the decision, not the meeting agenda.

**What are your priorities for next week?** — pull forward-looking signals: action items assigned to the user in Fathom summaries, "next steps" language in Basecamp comments, Linear issues in `In Progress` or `Todo` that the user owns, draft Gmail threads awaiting reply. Be specific — each bullet should be something the user could check off. If forward-looking data is thin, write a placeholder `* [add priorities]` and tell the user in the chat which areas you couldn't find priorities for.

**What are your biggest challenges right now?** — look for signals in transcripts and comments: blockers the user raised in meetings, unresolved threads in Gmail, Linear comments containing words like "blocked", "stuck", "waiting on". If nothing surfaces, leave a placeholder `* [add challenges, if any]` — do not fabricate challenges.

**What lessons did you learn? What opportunities do you see for improvement?** — leave as `* [your reflection]`. Do not invent lessons from connector data; this is reflective and belongs to the user. If a meeting transcript contains an explicit retro-style observation the user voiced, surface it as a candidate the user can keep or delete.

**Any kudos that you'd like to give?** — scan for teammates whose names appear positively in the user's outputs (e.g., "thanks to X for the review", "X unblocked me on Y"). Surface candidates as `* [candidate: thank @Name for ...]` and let the user accept or rewrite. Do not fabricate kudos.

### 4. Deliver

Present the filled-in template inline in chat as markdown unless the user asks for a file. Do not post it to Basecamp or anywhere else automatically.

After the template, add a short "Sources" footer listing the connectors queried and any that were unavailable. Example: `Pulled from: Fathom, Gmail, Drive, Basecamp, Linear. Not connected: Slack, Teams, Zoom.`

Then ask the user:
- Adjust the week?
- Want me to save it as a file?
- Want me to fill in the reflective sections from a few notes you give me?

## Notes and constraints

- The template is fixed. Do not add markdown headers (`#`, `##`) inside it — the user pastes it into Basecamp which uses different formatting.
- Bullets use `*` followed by a space, matching the template the user pastes from.
- For the three reflective sections (feelings, lessons, kudos), prefer empty placeholders over invented content. The 15-5 loses its value if Claude makes up the user's inner life.
- Cite sources inline using `[Title](URL)` format at the end of each bullet that came from a connector.
- Quote at most one sentence from any source document or transcript.
- If two items reference the same underlying work, merge them into one bullet.
- If the user wants the file saved, write it to the outputs directory as `15-5-week-of-YYYY-MM-DD.md`.
