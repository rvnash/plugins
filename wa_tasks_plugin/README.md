# wa_tasks_plugin

Foundation plugin for the rich's work at West Arete. Assumes a baseline set of connectors and includes a 15-5 weekly report, create-todos skill, and the Novo weekly readout.

## Bundled connectors

The plugin's `.mcp.json` configures five MCP servers. Installing the plugin offers all five to the user; each one still requires OAuth on first use.

| Tool         | Source                                                                                          | Used for                              |
| ------------ | ----------------------------------------------------------------------------------------------- | ------------------------------------- |
| Linear       | `https://mcp.linear.app/mcp` (OAuth on first use)                                               | Engineering tickets, projects, cycles |
| Google Drive | `https://drivemcp.googleapis.com/mcp/v1` (OAuth on first use)                                   | Documents the user edits              |
| Gmail        | `https://gmailmcp.googleapis.com/mcp/v1` (OAuth on first use)                                   | Important email threads               |
| Fathom       | `https://api.fathom.ai/mcp` (OAuth on first use)                                                | Recorded meetings, including CWSL     |
| Basecamp     | Local stdio server from [georgeantonopoulos/Basecamp-MCP-Server](https://github.com/georgeantonopoulos/Basecamp-MCP-Server) | Project posts, todos, comments        |

### One-time Basecamp setup

Basecamp has no public OAuth MCP, so each user installs the local server once. Run these commands on the machine where Cowork is installed:

```bash
git clone https://github.com/georgeantonopoulos/Basecamp-MCP-Server.git
cd Basecamp-MCP-Server
uv venv --python 3.12 venv
source venv/bin/activate
uv pip install -r requirements.txt
uv pip install mcp
python oauth_app.py   # open http://localhost:8000 and complete the Basecamp OAuth flow
```

Then set these environment variables so the plugin's `.mcp.json` can launch the server:

| Variable                  | Value                                                          |
| ------------------------- | -------------------------------------------------------------- |
| `BASECAMP_MCP_PYTHON`     | Absolute path to `Basecamp-MCP-Server/venv/bin/python`         |
| `BASECAMP_MCP_SCRIPT`     | Absolute path to `Basecamp-MCP-Server/basecamp_fastmcp.py`     |
| `BASECAMP_CLIENT_ID`      | Basecamp OAuth client ID (from your Basecamp integrations page) |
| `BASECAMP_CLIENT_SECRET`  | Basecamp OAuth client secret                                   |
| `BASECAMP_ACCOUNT_ID`     | Numeric account ID from your Basecamp URL                      |
| `BASECAMP_USER_AGENT`     | A descriptive string, e.g. `"company-base (you@example.com)"`  |

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

0.4.0
