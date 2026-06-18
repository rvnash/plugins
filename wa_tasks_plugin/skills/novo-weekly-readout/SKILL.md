---
name: novo-weekly-readout
description: "Draft a weekly Novo readout and post it as a draft message on the Readouts Basecamp message board for review. Use this skill whenever the user asks for a \"Novo weekly readout,\" \"weekly readout,\" \"Novo update,\" \"weekly update for Novo,\" \"Novo status post,\" or anything that sounds like compiling the week's Novo progress into a Basecamp post. Trigger even if the user only says \"write the readout\" or \"let's do this week's readout\" — the Novo readout is the default reading for those phrases."
---

# Novo Weekly Readout

Compile a weekly Novo readout as a .md text which can be copied/pasted into Basecamp.

## Fixed inputs

- Basecamp Project ID: `26016885`
- Basecamp Message Board ID: `10006186557`
- Linear workspace slug: `westarete-cwsl`
- Linear team: `Engineering` (key `ENG`)

## Workflow

Run the steps in order. Do not skip ahead; each step's output feeds the next.

### Step 1 — Compute the date range

The readout covers **last Friday through right now**.

- Determine the current date/time in the America/New_York timezone
- "Last Friday" = the most recent Friday that is strictly before today. If today is Friday, use the Friday from the prior week (7 days ago), not today.
- Store as `since_date` (ISO 8601, e.g. `2026-05-15T00:00:00`).
- State the resolved range to the user in one line so they can correct it. Example: "Covering Fri May 15 → Wed May 20."

### Step 2 — Read the 3 most recent readouts from the Basecamp Message Board

Goal: extract the template structure (section headings, ordering, tone, length, formatting conventions like bold/links/checklists).

Use **whichever Basecamp MCP server is connected** — tool names vary between servers, so match by capability (the Basecamp tools are usually prefixed `basecamp`). If no Basecamp tool is connected, stop and tell the user no Basecamp connector is available.

Try these in order, stopping at the first one that works:

1. **List messages on a board** — call the connected server's "list messages" tool (e.g. `list_messages` / `get_messages`) with the Project ID and Message Board ID from above.
2. **Single-message fetch** — if you can identify the 3 most recent message IDs from any other tool result (e.g. a smaller-scoped search), call the server's "get message" tool (e.g. `get_message`) for each.

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
3. Screenshots or videos to put place holders in for. Place a placeholder line like `[INSERT: mixpanel-usage-2026-05-20.png]` exactly where the image should go).
4. Any problems to highlight
5. Any bug fixes to highlight
6. Anything from the auto-gathered list (Step 3.5) to **exclude** from the draft — free text or "none."


### Step 5 — Compose the draft

Mirror the template from Step 2 as closely as possible: same section order, same heading style, comparable length per section.

- **Subject**: follow the past subject pattern (most readouts use a date or week label).
- **Body**: use Basecamp's .md conventions. Do not invent sections that weren't in the past 3 readouts; do not drop sections that were consistent across them.
- Sections are delimited by bold and blue headings
- Insert image placeholders inline where screenshots/videos/Mixpanel belong.
- Do not use Linear Ticket numbers, they are internal only
- Do not create links to the source information
- Weave in items from Step 3.5 alongside Linear tickets. For Fathom/Zoom meeting takeaways, link the meeting; for Slack/Teams items, link the message; for Gmail items, link the thread. Mark anything you're unsure belongs in the Novo bucket with a trailing `(?)` for the user to verify when they open the posted draft in Basecamp.

### Step 6 — Present the result as .md text to the user


## Output contract

The skill's final response to the user must contain, in this order:

1. The resolved date range, one line.
2. The subject line that was posted.
3. The report readout in .md

Nothing else.
