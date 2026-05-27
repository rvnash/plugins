---
name: create-todos
description: Read back Rich's most recent 15-5 from Basecamp and create a Basecamp todo for every "priority for next week" item in the Misc Todos list of the 🏠 Rich Nash project. Use this skill when the user says "create todos", "create todos from my 15-5", "make todos for next week", "turn my priorities into todos", "add my next-week priorities to Basecamp", or any variation of converting 15-5 priorities into Basecamp todos. Trigger even when the user only says "/create-todos".
---

# create-todos

Read back Rich's most recently posted 15-5 and create one Basecamp todo per "priority for next week" item in the Misc Todos list of the 🏠 Rich Nash project.

This skill is the natural follow-on to `15-5-report`: the 15-5 produces a list of priorities, and this skill puts each priority on a tracked todo list so it doesn't disappear in a Basecamp comment.

## Fixed locations

These IDs are stable for Rich's account and should be used as-is:

- 15-5 message: https://3.basecamp.com/4206247/buckets/25906397/messages/9428630577
  - `project_id`: `25906397`
  - `recording_id` (the message): `9428630577`
- Target todolist: "Misc Todos" in the 🏠 Rich Nash project
  - `project_id`: `25906397`
  - `todolist_id`: `5377896272`
  - App URL: https://3.basecamp.com/4206247/buckets/25906397/todolists/5377896272

## Workflow

### 1. Fetch the most recent 15-5

Call `mcp__basecamp__get_comments` with `project_id="25906397"` and `recording_id="9428630577"`. Basecamp returns comments in chronological order; the most recent 15-5 is the **last** comment in the response (or the first when sorted by `created_at` descending). Pick the one whose author is Rich Nash and whose content has the 15-5 section headers.

If no comments exist on the message, stop and tell the user: "No 15-5 has been posted yet — run /15-5-report first."

### 2. Read it back to the user

Convert the comment's HTML body back to readable markdown and print it in chat as a fenced block, preceded by a one-line header like `Most recent 15-5 (posted YYYY-MM-DD HH:MM):`. Strip the `<div>`, `<strong>`, `<ul>`, `<li>`, and `<a>` tags; keep the link URLs as `[Title](URL)`; preserve the section question lines as bold markdown (`**…**`).

### 3. Extract the priorities

Find the section under the `What are your priorities for next week?` header. Each `<li>` inside that section's `<ul>` becomes one todo. Strip surrounding whitespace and any leading bullet marker. Skip any bullet whose text is a placeholder like `[add priorities]` or that is empty.

### 4. Create todos

For each priority bullet, call `mcp__basecamp__create_todo` with:

- `project_id`: `"25906397"`
- `todolist_id`: `"5377896272"`
- `content`: the bullet text as plain text (strip HTML tags, but preserve URLs by appending them in parentheses if they were inside an `<a>` — e.g. `Coordinate prod push of First Year Section Assignment (https://linear.app/...)`)
- `description`: omit unless the bullet had additional sentences beyond the first; in that case put the extra sentences here in HTML
- `due_on`: Try to discern the due date from the context. But, unless you're sure, set the `due_on date` to the Monday following the 15-5.
- `assigned_to`: Assign all todo's to me, Rich Nash

Make these calls sequentially, not in parallel, so a failure on one doesn't orphan the others.

### 5. Report back

After all todos are created, print a short summary in chat:

```
Created N todos in Misc Todos:
- <todo text> — <app_url>
- ...
```

Use the `app_url` from each create_todo response so the user can click straight to each todo. End with a single link to the todolist itself: https://3.basecamp.com/4206247/buckets/25906397/todolists/5377896272

If any todo failed to create, list it separately under a `Failed:` heading with the error message.

## Notes and constraints

- Always pull the **latest** 15-5 comment, never an older one, unless the user names a specific date.
- Never create duplicate todos. Before creating, call `mcp__basecamp__get_todos` with `todolist_id=5377896272` and skip any priority whose content already matches an open todo's content (case-insensitive substring match on the first 60 characters is enough).
- Do not modify the 15-5 comment.
- Do not mark any todos as complete.
- If the user asks to create todos for a *different* week, look further back in the comments and pick the comment matching that `Week of YYYY-MM-DD` header.
- If the priorities section is missing or empty, stop and tell the user — do not invent todos.

## Required tools

- `mcp__basecamp__get_comments` — fetch comments on the 15-5 message
- `mcp__basecamp__get_todos` — dedupe check
- `mcp__basecamp__create_todo` — create each todo
