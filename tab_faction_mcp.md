---
title: faction_mcp
layout:  null
tab: true
order: 1
tags: mcp, reporting, vulnerability management, pentesting, ai-cyber-security, agentic, application-security
---

## Faction MCP Server
![image](/www-project-faction/assets/images/faction-opencode.png)

The Faction MCP server is a Model Context Protocol (MCP) server that exposes a Faction instance — assessments, vulnerabilities, retests, vulnerability templates, and audit logs — to any MCP-compatible AI client (Claude Code, Claude Desktop, OpenCode, GitHub Copilot CLI, LM Studio, and others).

It lets an AI agent read from and write to Faction directly, so workflows like running a CLI tool, parsing its output, creating a finding, and generating a report can be automated end-to-end without copying data between tools by hand.

Source repository: <https://github.com/factionsecurity/faction-mcp> (MIT licensed)

## What you can do with it

- Drive Faction from an AI client: pull your assessment queue, open vulnerabilities, retests, and audit log entries.
- Create and update vulnerabilities programmatically, including from default templates.
- Generate or download assessment reports (PDF/DOCX) and have the client open them directly.
- Generate executive summaries from stripped-down vulnerability data optimized for LLM context windows.
- Wire CLI security tools (nmap, nuclei, custom scripts, etc.) into reproducible workflows by combining the MCP server with agent skill files.
- Use any AI provider, including local LLMs (LM Studio, Ollama-style runners), so assessment data does not have to leave your environment.

## Requirements

- A running Faction instance.
- A Faction API key — generated from your user profile page in Faction.
- Docker, Docker Compose, Podman, or Podman Compose on the host that will run the MCP server.
- An MCP-compatible client (Claude Code, Claude Desktop, OpenCode, Copilot CLI, LM Studio, etc.).

## Installation

There are two supported installation paths.

### Option 1: Docker Desktop MCP Catalog

Install directly from the Docker Desktop MCP Catalog and enter `FACTION_API_KEY` and `FACTION_BASE_URL` when prompted. This is the lowest-friction option if you already use Docker Desktop.

### Option 2: Docker Compose (Docker or Podman)

Works anywhere Docker Compose or Podman Compose is available.

**1. Clone the repository and create the environment file**

```bash
git clone https://github.com/factionsecurity/faction-mcp.git
cd faction-mcp
cp .env.example .env
```

**2. Edit `.env` with your Faction details**

```
FACTION_API_KEY=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
FACTION_BASE_URL=https://faction.yourcompany.com
```

To rotate credentials later, edit `.env` — no rebuild is required.

**3. Build the image**

```bash
docker compose build
# or
podman-compose build
```

**4. Configure your MCP client**

Add the server to your client's MCP config. Use the absolute path to your cloned `docker-compose.yml`.

Claude Code / Claude Desktop (`~/.claude/settings.json` or `claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "faction": {
      "command": "docker",
      "args": [
        "compose",
        "-f", "/absolute/path/to/faction-mcp/docker-compose.yml",
        "run", "--rm", "-T", "faction-mcp"
      ]
    }
  }
}
```

OpenCode:

```json
{
  "mcp": {
    "faction": {
      "type": "local",
      "command": [
        "docker", "compose",
        "-f", "/absolute/path/to/faction-mcp/docker-compose.yml",
        "run", "--rm", "-T", "faction-mcp"
      ],
      "enabled": true
    }
  }
}
```

Podman users substitute `podman-compose` for `docker compose`. The `-T` flag disables pseudo-TTY allocation so stdio passes cleanly between the client and the server.

After restarting the client, run its MCP listing command (`/mcps` in OpenCode, equivalent elsewhere) to confirm `faction` is connected.

### Shared reports / images directory

The compose files mount one host directory into the container at `/app/reports`. By default this is `/tmp`. It is used for two things:

- **Report downloads.** `get_assessment_report` and `generate_assessment_report` write the PDF/DOCX here and return the host path so the client can open the file.
- **Image uploads.** `upload_assessment_image` reads from this directory. The agent is instructed to copy or save images into it first, then pass the host path. The server translates `/tmp/foo.png` → `/app/reports/foo.png` internally.

Routing image bytes through disk avoids round-tripping large base64 strings through tool arguments, where token-stream drift can corrupt the payload.

To use a different host folder, set `FACTION_REPORTS_HOST_DIR` to an absolute path in `.env`:

```
FACTION_REPORTS_HOST_DIR=/Users/me/faction-reports
```

## Available Tools

The server exposes the following tool surface to the AI client.

### Assessments

| Tool | Description |
|------|-------------|
| `get_assessment_queue` | Active queue for the authenticated user (in-progress / upcoming / past-due). Use for "my recent assessments". |
| `get_completed_assessments` | Completed assessments in a date range with full detail. |
| `get_completed_assessments_condensed` | Same as above with large text blocks stripped. Preferred for stats and historical summaries. |
| `get_assessment` | Full details for a specific assessment by ID. |
| `update_assessment` | Update assessment notes, executive summary, distribution list, custom fields. |
| `get_assessment_vulnerabilities` | Full vulnerability data for an assessment (large; includes HTML and screenshots). |
| `get_vulnerability_summary_data` | Stripped vulnerability data optimized for executive summary generation. |
| `get_assessment_report` | Download the existing PDF/DOCX report to the configured reports directory. |
| `generate_assessment_report` | Kick off a fresh report build and poll for completion. |
| `check_report_status` | Standalone poll used to resume waiting when generation outlasts the initial wait. |

### Vulnerabilities

| Tool | Description |
|------|-------------|
| `get_vulnerabilities` | All vulnerabilities opened in a date range with full detail. |
| `get_vulnerabilities_condensed` | Same with large text blocks stripped. |
| `create_vulnerability` | Add a vulnerability to an assessment. Interactively prompts for missing fields and offers matching templates. |
| `update_vulnerability` | Update fields on an existing vulnerability. |
| `add_templated_vulnerability` | Add a vulnerability from a default template. |
| `get_vulnerability` | Get a vulnerability by ID. |
| `get_vulnerability_by_tracking` | Get a vulnerability by tracking ID (e.g. Jira ticket). |
| `set_vulnerability_tracking` | Assign a tracking ID. |
| `set_vulnerability_status` | Set remediation status (dev/prod closed dates). |
| `get_risk_levels` | Configured risk level definitions; unmapped slots filtered out. |
| `get_categories` / `get_category` / `create_category` | Manage vulnerability categories (create requires manager role). |

### Vulnerability Templates

| Tool | Description |
|------|-------------|
| `get_vulnerability_templates` | All default vulnerability templates. |
| `search_vulnerability_templates` | Search templates by name (partial match). |
| `get_vulnerability_template` | Get a template by ID. |
| `create_vulnerability_templates` | Create or update default templates. |

### Retests / Verifications

| Tool | Description |
|------|-------------|
| `get_verification_queue` | Retest queue for the authenticated user. |
| `get_all_verifications` | All verifications, optionally filtered by date range. |
| `get_user_verifications` | Verifications for a specific user. |
| `complete_verification` | Mark a retest as passed or failed. |
| `schedule_retest` | Schedule a retest for a vulnerability. |

### Audit Logs

| Tool | Description |
|------|-------------|
| `get_audit_log` | System audit log for a date range (admin role required). |
| `get_assessment_audit_log` | Audit entries for all assessments in a date range. |
| `get_assessment_audit_log_by_id` | Audit entries for a specific assessment. |
| `get_user_audit_log` | Audit entries for a specific user. |

## Key Workflows

### Creating a vulnerability

`create_vulnerability`, `update_vulnerability`, and `add_templated_vulnerability` accept severity, impact, and likelihood as risk-level **names** (`"Critical"`, `"High"`, `"P1"`, etc.). The server resolves the name to the correct numeric ID for your Faction instance via `get_risk_levels`, so the LLM never needs to know the IDs and skills do not break when IDs differ between instances.

When required information is missing, `create_vulnerability` walks the user through an interactive workflow via MCP elicitation (supported by Claude Code, Claude Desktop, and other elicitation-capable clients):

1. **Title and severity** — if either is missing, the user is prompted. Risk levels are presented as a dropdown of the names actually configured on the instance.
2. **Template offer** — if no description, recommendation, or template ID is supplied, the server searches default templates by title and offers matches. Selecting one auto-populates description and recommendation.
3. **Mirror severity** — if severity is set but impact/likelihood are not, the server asks whether to mirror severity to both, or to set them explicitly.

If the client does not support elicitation, the tool returns an error listing the missing fields so the agent can ask the user in plain text.

### Generating executive summaries

Use `get_vulnerability_summary_data` rather than `get_assessment_vulnerabilities`. It returns text-only, stripped vulnerability data optimized for LLM processing. After the AI drafts the summary HTML, it calls `update_assessment` to write it back to Faction.

### Generating and downloading reports

Three tools cover report handling:

- `generate_assessment_report` — starts a fresh report build (use `retest=true` for finalized assessments) and polls up to `max_wait_seconds` (default 60).
- `check_report_status` — standalone poll when the initial wait expires. Pass `last_known_gentime` from the generate response.
- `get_assessment_report` — downloads the existing report. The file is written to the configured reports directory and the absolute host path is returned as `file_path`.

A typical run: ask the agent to generate a new report for an assessment ID; the server fires generation, waits, and reports completion; the agent then calls `get_assessment_report` and surfaces the file path.

## Automating CLI Tools with Agent Skills

The MCP server gives the agent access to Faction. To turn ad-hoc agent runs into reproducible workflows, pair it with agent skill files (`SKILL.md` / `AGENTS.md`) that describe a fixed procedure: which command to run, how to parse the output, what severity to use, what fields to populate in Faction.

A typical skill for an nmap-driven information finding might:

1. Require explicit user authorization that the target is in scope.
2. Run a fast top-ports scan (`nmap -Pn --top-ports 100 <target>`).
3. Run a vulnerability scan against only the discovered open ports (`nmap -sV --script=vuln -p <ports> <target>`).
4. Parse the output into a list of ports, services, versions, and identified CVEs.
5. Call `create_vulnerability` with a fixed naming convention, the configured lowest severity tier (e.g. `"Recommended"`), and a markdown details field containing two tables (commands run, and ports/weaknesses).

![image](/www-project-faction/assets/images/mcp-namp-faction.png)

Because the skill pins the procedure — command flags, severity, description structure, table format — the resulting findings are consistent across assessments and across whoever (or whatever agent) runs the skill. The same pattern applies to any CLI tool that produces parseable output.

A worked nmap skill example is published with the Faction blog post linked below.

## Reference

- Repository and README: <https://github.com/factionsecurity/faction-mcp>
- Walkthrough with full nmap skill example: [Automate Your Pentesting Workflow: Connecting Faction to AI Agents via MCP](https://medium.com/@we-are-faction/automate-your-pentesting-workflow-connecting-faction-to-ai-agents-via-mcp-c985c8c79e2d)
- Faction core project: <https://github.com/factionsecurity/faction>
- Background on agent skills: <https://agents.md/>
