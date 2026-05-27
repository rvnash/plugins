# wa_tasks_plugin

Foundation plugin for the rich's work at West Arete. Assumes a baseline set of connectors and includes a 15-5 weekly report, create-todos skill, and the Novo weekly readout.

## Bundled connectors

The plugin's `.mcp.json` configures four MCP servers. Installing the plugin offers all four to the user; each one still requires OAuth on first use.

| Tool         | Source                                                                                          | Used for                              |
| ------------ | ----------------------------------------------------------------------------------------------- | ------------------------------------- |
| Linear       | `https://mcp.linear.app/mcp` (OAuth on first use)                                               | Engineering tickets, projects, cycles |
| Google Drive | `https://drivemcp.googleapis.com/mcp/v1` (OAuth on first use)                                   | Documents the user edits              |
| Gmail        | `https://gmailmcp.googleapis.com/mcp/v1` (OAuth on first use)                                   | Important email threads               |
| Fathom       | `https://api.fathom.ai/mcp` (OAuth on first use)                                                | Recorded meetings, including CWSL     |

## Basecamp (bring your own connector)

Basecamp is **not bundled** in this plugin's `.mcp.json`. The `create-todos`, `15-5-report`, and `novo-weekly-readout` skills use **whatever Basecamp MCP server happens to be connected** in your environment — they match Basecamp tools by capability (list comments, list todos, create todo, list/get/create message, search) rather than by a hard-coded tool name, so any Basecamp MCP works.

Connect a Basecamp MCP server however you prefer (a hosted Basecamp connector, or a local stdio server such as [georgeantonopoulos/Basecamp-MCP-Server](https://github.com/georgeantonopoulos/Basecamp-MCP-Server)). If no Basecamp connector is present, the Basecamp-dependent skills will tell you so and stop rather than fail mid-run.

## Included skills

### `15-5-report`

Draft a 15-5 weekly report by pulling this week's activity from connected tools and filling in the West Arete 15-5 template. Triggered by phrases like "15-5", "fifteen-five", "weekly 15-5".

### `create-todos`

Read back Rich's most recent 15-5 from Basecamp and create one tracked todo per "priority for next week" item in the Misc Todos list of the 🏠 Rich Nash project. Natural follow-on to `15-5-report`. Triggered by phrases like "create todos", "create todos from my 15-5", or `/create-todos`.

### `novo-weekly-readout`

Draft the weekly Novo readout as a Basecamp post for review (does not publish). Pulls completed Linear issues since last Friday, mirrors the format of recent readouts, and prompts the user for accomplishments, in-progress items, screenshots, problems, bug fixes, and Mixpanel usage. Triggered by phrases like "Novo weekly readout", "Novo update", "write the readout".

## Adding skills

Drop new skill folders under `skills/` following the same pattern as the included skills. Each skill needs a `SKILL.md` with YAML frontmatter (`name`, `description`) and an imperative body written for Claude to follow.

## Version

0.5.0
