---
name: 15-5-report
description: Draft a 15-5 weekly report for the user by pulling this week's activity from connected tools (Fathom, Gmail, Google Drive, Basecamp, Linear, Slack, Teams, Zoom — whichever are connected) and filling in the West Arete 15-5 template. Use this skill whenever the user asks for a "15-5", "fifteen-five", "15/5 report", "weekly 15-5", "my 15-5 for this week", or any phrasing involving the 15-5 ritual. Trigger even when the user says only "do my 15-5" or "write the 15-5" — this is the default reading for those phrases. Do not use this skill for a generic "Friday status" or "weekly readout" — those have their own skills with different templates.
---

# 15-5 report

Draft a 15-5 weekly report for the user. The 15-5 is a West Arete ritual based on Yvon Chouinard's Patagonia practice: a weekly write-up that takes about fifteen minutes to write and five minutes to read. It uses a fixed six-section template and is posted for the user's manager at end of week.

The goal of this skill is to do the heavy data-gathering across connectors so the user can spend their fifteen minutes on the reflective parts (feelings, lessons, kudos) rather than on remembering what they did.

## Where the report goes

The finished 15-5 is posted as a **reply (comment) on a fixed Basecamp message thread** — the same thread `create-todos` reads back from:

- Thread: https://3.basecamp.com/4206247/buckets/25906397/messages/9428630577
- `project_id`: `25906397`
- `recording_id` (the message being replied to): `9428630577`

Posting uses whichever Basecamp MCP server is connected (tool names vary by server — match by capability). The operation needed is **create a comment on a recording/message** (e.g. `create_comment`), passing `project_id=25906397`, `recording_id=9428630577`, and the HTML `content`. If no Basecamp connector is available, fall back to presenting the report inline and tell the user it could not be posted.

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

The report follows this exact structure and ordering. Do not rename, reorder, or add sections. Each question line is **bold**.

The _italicized_ line under some questions is **writer guidance** carried over from the source West Arete template. Read it and let it shape what you write — but it is NOT part of the report. Strip every guidance line from the output: it must not appear in the inline preview or the posted Basecamp reply. Output only the bold question and its answer.

```
Week of YYYY-MM-DD

**How did you feel at work this week?**

*

**What did you accomplish this week?**
_List your completed activities and accomplishments. What is working well?_

*

**What are your priorities for next week?**
_Be specific._

*

**What are your biggest challenges right now?**
_And how might we help?_

*

**What lessons did you learn? What opportunities do you see for improvement?**
_These may be lessons for yourself, or insights that might benefit other parts of the company. What questions are you trying to solve? Any ideas for how to improve the company?_

*

**Any kudos that you'd like to give?**
_Appreciate teammates by sharing the impact they had. If appropriate, mention how their actions aligned with our company values._

*
```

The `Week of YYYY-MM-DD` line is the Monday of the week and is NOT bold. The italicized guidance lines are never rendered in the output. When the report is posted to Basecamp (see Deliver), render it as HTML: each bold question as `<strong>…</strong>` on its own line, and each section's answer as a `<ul><li>…</li></ul>` list (or a single `<div>` line if there's only one). Preserve inline source links as `<a href="URL">Title</a>`.

## Workflow

### 1. Determine the week

Compute the Monday of the current week and use it as the "Week of" date. The reporting window is Monday 00:00 through today (or through Friday if today is later). Resolve to ISO dates and pass those literal dates to connector tools — never pass relative phrases like "this week".

If the user explicitly names a different week, honor that and recompute the Monday.

### 2. Gather activity in parallel

Make these calls as a single batch when the tool harness allows it:

**Fathom** — `list_meetings` or `search_meetings` bounded to the window. For each meeting, prefer `get_meeting_summary` over `get_meeting_transcript`. Capture title, date, attendees, recording URL, decisions, action items assigned to the user, and outcomes.

**Gmail** — `search_threads` with date-bounded queries. Run at least two queries: (a) threads where the user sent a substantive reply (label `SENT`, date-bounded), and (b) starred threads in the window. Skip routine notifications and calendar invites.

**Google Drive** — `list_recent_files` filtered to items the user modified in the window. Capture title, link, and a one-line "what changed" note if the doc is short enough to read.

**Basecamp** — use whichever Basecamp MCP server is connected (tool names vary by server; match by capability — its tools are usually prefixed `basecamp`). For each project the user participates in, use the server's activity/events tool scoped to the window if it has one, otherwise its search tool (e.g. a `search` tool) and its list tools for recordings/todos/messages. Capture posts the user authored, todos the user completed, and substantive comments the user left. If no Basecamp tool is connected, skip Basecamp and note it in the Sources footer.

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

### 4. Review, then post as a reply

1. **Preview inline.** Present the filled-in template in chat as markdown (questions in bold via `**…**`) so the user can read it before anything is posted. Below it, add a short "Sources" footer listing the connectors queried and any unavailable. Example: `Pulled from: Fathom, Gmail, Drive, Basecamp, Linear. Not connected: Slack, Teams, Zoom.`

2. **Flag the gaps.** Call out the reflective sections that still need the user (feelings, lessons, kudos) and any sections left as placeholders (priorities, challenges). Offer to fill them from a few notes the user gives you.

3. **Confirm before posting.** Ask the user to confirm (and to adjust the week, or supply reflective content) before anything goes to Basecamp. Do not post until the user says to.

4. **Post as a reply.** On confirmation, convert the report to Basecamp HTML per "The template" (bold questions as `<strong>`, answers as `<ul><li>` lists, source links as `<a href>`), then call the connected Basecamp server's "create comment" tool with `project_id=25906397`, `recording_id=9428630577`, and the HTML `content`. This adds the 15-5 as a reply on the fixed thread.

5. **Return the link.** Capture the `app_url` (or `url`) from the create-comment response and give it to the user as `Posted: <url>` so they can open and edit the reply in Basecamp.

If the user only wants a file instead of posting, write it to the outputs directory as `15-5-week-of-YYYY-MM-DD.md` and skip the post.

## Notes and constraints

- The template is fixed: same sections, same order, bold questions. The italicized guidance lines stay in the template to steer your writing but are stripped from the output (preview and posted reply).
- In the inline preview, questions are bold markdown (`**…**`) and answers use `*` bullets. In the posted Basecamp reply, questions are `<strong>` and answers are `<ul><li>` lists — never markdown headers (`#`, `##`).
- For the three reflective sections (feelings, lessons, kudos), prefer empty placeholders over invented content. The 15-5 loses its value if Claude makes up the user's inner life.
- Cite sources inline using `[Title](URL)` format at the end of each bullet that came from a connector.
- Quote at most one sentence from any source document or transcript.
- If two items reference the same underlying work, merge them into one bullet.
- If the user wants the file saved, write it to the outputs directory as `15-5-week-of-YYYY-MM-DD.md`.
