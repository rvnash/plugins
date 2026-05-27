---
name: novo-weekly-readout
description: "Draft a weekly Novo readout and post it as a draft message on the Readouts Basecamp message board for review. Use this skill whenever the user asks for a \"Novo weekly readout,\" \"weekly readout,\" \"Novo update,\" \"weekly update for Novo,\" \"Novo status post,\" or anything that sounds like compiling the week's Novo progress into a Basecamp post. Trigger even if the user only says \"write the readout\" or \"let's do this week's readout\" — the Novo readout is the default reading for those phrases."
---

# Novo Weekly Readout

Compile a weekly Novo readout, post it to the Readouts Basecamp message board, and return the link to the posted message so the user can review and edit it in Basecamp.

## Fixed inputs

- Basecamp project ID: `38088674`
- Basecamp **Readouts** message board ID (source for past readouts to mirror): `7950419793` — if calls against this ID fail, also try `7546621971` (the ID returned by `get_message_board` for the same project)
- Basecamp Readouts board URL (for reference): https://3.basecamp.com/4206247/buckets/38088674/message_boards/7950419793
- Linear workspace slug: `westarete-cwsl`
- Linear team: `Engineering` (key `ENG`)

## Workflow

Run the steps in order. Do not skip ahead; each step's output feeds the next.

### Step 1 — Compute the date range

The readout covers **last Friday through right now**.

- Determine the current date/time in the user's timezone (default America/New_York if unknown).
- "Last Friday" = the most recent Friday that is strictly before today. If today is Friday, use the Friday from the prior week (7 days ago), not today.
- Store as `since_date` (ISO 8601, e.g. `2026-05-15T00:00:00`).
- State the resolved range to the user in one line so they can correct it. Example: "Covering Fri May 15 → Wed May 20."

### Step 2 — Read the 3 most recent readouts on the board

Goal: extract the template structure (section headings, ordering, tone, length, formatting conventions like bold/links/checklists).

Try these in order, stopping at the first one that works:

1. **Basecamp MCP** — call `basecamp:get_messages` with `project_id=38088674`, `message_board_id=7950419793` (the **Readouts** board, not the Readouts board). If "Tool result too large," retry with `message_board_id=7546621971`. If still too large, fall through.
2. **Basecamp MCP, single-message fetch** — if you can identify the 3 most recent message IDs from any other tool result (e.g. a smaller-scoped search), call `basecamp:get_message` for each.

From the 3 posts, derive a template: subject line pattern (e.g. "2026-05-16 Novo Weekly Readout"), section headers in order, what kind of content goes in each section, and any standard sign-off.

### Step 3 — Pull Linear issues completed in the window

Use `Linear:list_issues` with:

- `statusType: "completed"`
- `includeArchived: true`
- Keep only issues with `completedAt >= since_date`.

For each kept issue, capture: identifier (e.g. `ENG-1738`), title, description.

If the list is empty after filtering, ask the user before continuing — they may want to widen the filter or check a different team.

### Step 3.5 — Pull additional accomplishments and Novo news from other sources

Goal: capture Novo-related accomplishments, decisions, customer signals, and news from the rest of the user's tools during the same window (`since_date` → now). Run each source that has a connector available; skip silently (do not error) if a connector is missing.

Filter every result to Novo-relevant content. Heuristics: mentions of "Novo," any ENG ticket identifier, Novo customer/stakeholder names, Novo project channels, or Novo Basecamp project mentions. When in doubt, include with a `(?)` marker so the user can confirm when reviewing the posted draft in Basecamp.

Run these in parallel where possible:

1. **Fathom** — call `fathom:list_meetings` (or `fathom:search_meetings`) for meetings with `start_time >= since_date`. Keep meetings whose title, attendees, or summary reference Novo. For each kept meeting, call `fathom:get_meeting_summary` and extract: decisions, action items, customer feedback, and anything that reads as an accomplishment or news item. Record meeting title, date, and a one-line takeaway with the Fathom URL.
2. **Microsoft Teams** — search Teams messages and channels for Novo mentions in the window. Capture decisions, announcements, blockers resolved, and customer questions. Record channel/chat name, date, author, and a one-line summary with a link.
3. **Slack** — call `slack:slack_search_public_and_private` (fall back to `slack_search_public` if the private search isn't available) with queries like `Novo after:<since_date>`, `ENG- after:<since_date>`, and any known Novo channel names. Also `slack_read_channel` on dedicated Novo channels if known. Capture announcements, shipped-it messages, customer reports, and decisions. Record channel, date, author, permalink, and a one-line summary.
4. **Gmail** — call `gmail:search_threads` with queries like `Novo after:<since_date>`, plus searches for known Novo stakeholder domains/addresses. Keep threads that contain product news, customer feedback, contract/legal updates, or release coordination. Record subject, date, sender, and a one-line summary with the thread link.
5. **Zoom** — call `zoom:search_meetings` / `zoom:recordings_list` for recordings in the window. Keep meetings whose topic or participants reference Novo. For each, pull the summary or transcript snippet and extract Novo-relevant takeaways. Record meeting topic, date, and a one-line summary with the recording link.

Collate the findings into a single working list grouped by source, deduplicated against the Linear issues from Step 3 (don't list a ticket twice if Slack/Teams just announced it shipping — merge them, keeping the ticket as the primary entry and the chat link as supporting context).

If a source returns nothing relevant, record "no Novo items found in <source>" so Step 4 can show the user what was already checked.

### Step 4 — Ask the user for the human inputs

Before asking, show the user a brief summary of what Step 3 and Step 3.5 already gathered (counts per source, e.g. "Linear: 7 tickets · Fathom: 2 meetings · Slack: 4 threads · Gmail: 1 thread · Teams: 0 · Zoom: 1 recording"), so they can fill only the gaps.

Use the `ask_user_input_v0` tool. Ask all of these in one call so the user answers once:

1. Other accomplishments **beyond what was found in Linear, Fathom, Teams, Slack, Gmail, and Zoom** — free text or "none."
2. Things in progress to highlight — free text or "none."
3. Screenshots or videos to include — ask the user to upload them in their next message, or paste links. Specify file types accepted.
4. Any problems to highlight
5. Any bug fixes to highlight
6. Mixpanel usage screenshot — ask for an upload or link, and which view/metric it represents.
7. Anything from the auto-gathered list (Step 3.5) to **exclude** from the draft — free text or "none."

After they respond, if they uploaded files, note the filenames/paths so you can reference them in the draft (Basecamp accepts inline images via paste; in the draft, place a placeholder line like `[INSERT: mixpanel-usage-2026-05-20.png]` exactly where the image should go).

### Step 5 — Compose the draft

Mirror the template from Step 2 as closely as possible: same section order, same heading style, comparable length per section.

- **Subject**: follow the past subject pattern (most readouts use a date or week label).
- **Body**: use Basecamp's HTML rich text conventions — `<h1>`, `<h2>`, `<ul>`, `<li>`, `<a href>`, `<strong>`, `<em>`, `<div>`, line breaks. Do not invent sections that weren't in the past 3 readouts; do not drop sections that were consistent across them.
- Sections are delimited by bold and blue headings
- Insert image placeholders inline where screenshots/videos/Mixpanel belong, using the filenames the user provided.
- Weave in items from Step 3.5 alongside Linear tickets. For Fathom/Zoom meeting takeaways, link the meeting; for Slack/Teams items, link the message; for Gmail items, link the thread. Mark anything you're unsure belongs in the Novo bucket with a trailing `(?)` for the user to verify when they open the posted draft in Basecamp.

### Step 6 — Post the draft to the Readouts board

Call `basecamp:create_message` with:

- `project_id`: `38088674`
- `message_board_id`: `7546621971` (the **Readouts** board)
- `subject`: the subject line composed in Step 5
- `content`: the HTML body composed in Step 5
- `status`: `draft`

Capture the `app_url` (or `url`) returned by `create_message` — that is the link the user will follow to review and edit the draft in Basecamp.

If `create_message` fails, do not retry against a different board. Report the error to the user, include the composed subject and HTML body in the response so nothing is lost, and stop.

Image placeholders (e.g. `[INSERT: mixpanel-usage-2026-05-20.png]`) remain inline in the posted HTML — the user will replace them with pasted images when editing in Basecamp.

### Step 7 — Return the link to the user

Present the result as:

1. The resolved date range, one line.
2. The subject line that was posted.
3. The link to the posted draft on the Readouts board (the `app_url` from Step 6), labeled clearly (e.g. `Draft posted: <url>`).
4. A short list of any image placeholders the user still needs to replace, with the matching filename for each.
5. A one-line review checklist (e.g. "Verify the 3 ENG tickets I flagged with (?) belong in the Novo bucket, then replace image placeholders.").

Do not paste the full HTML body or plain-text rendering into the chat response — the user will review and edit it directly in Basecamp via the link.

## Output contract

The skill's final response to the user must contain, in this order:

1. The resolved date range, one line.
2. The subject line that was posted.
3. The link to the posted draft on the Readouts board (Step 6 `app_url`), labeled clearly.
4. The image-placeholder list (filenames the user must paste in).
5. The review checklist (one short paragraph or short bullet list).

Nothing else. Do not paste the HTML body or plain-text rendering into chat — the user reviews and edits the draft in Basecamp via the link.
