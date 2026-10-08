# Captino plugin for Claude Code

Connects Claude Code to the Captino MCP server at `https://api.captino.io/mcp`
and adds skills for working with funnels, pages and leads.

## Install

This repository is both the plugin and its marketplace:

```shell
/plugin marketplace add Grumsen-Development-ApS/captino-claude-plugin
/plugin install captino@captino
```

Or test a local checkout without installing:

```bash
claude --plugin-dir .
```

On enable, the plugin asks for a **Captino MCP API key**. Create one in the
Captino app under **Settings → MCP** (`/app/mcp-settings`). Keys start with
`cap_mcp_`. Grant Funnel write, Contact write, Form write, Performance read and
Subscription read to use every skill. The key is stored in Claude Code's secure storage and
sent as `Authorization: Bearer <key>`.

Check the connection with `/mcp`; the server shows up as `captino`.

## Skills

| Skill | What it does | Needs |
| :-- | :-- | :-- |
| `/captino:funnel-performance` | Visits, opt-ins and opt-in rate for a period, change vs previous period, funnel ranking, head-to-head | Funnel read, Performance read |
| `/captino:funnel-dropoff` | Per-step conversion, biggest drop, per-question form abandonment | Funnel read, Performance read |
| `/captino:leads` | New leads, contact search, lead activity and source funnel, pipeline counts | Contact read |
| `/captino:lead-followup` | Qualify a lead, move it through the pipeline, set deal value, add a note, schedule a meeting | Contact write |
| `/captino:pipeline-report` | Weekly or monthly account overview: pipeline, funnels, subscription usage | Contact, Funnel, Performance, Subscription read |
| `/captino:page-builder` | Build, edit and publish funnel step pages (landing, opt-in, thank-you, popups) from a brief | Funnel write |
| `/captino:form-builder` | Build, edit and publish multi-question forms (lead, qualification, survey) and add them to a funnel as a form step | Form write, Funnel write |
| `/captino:captino-setup` | Create a key, configure the plugin, troubleshoot 401/403/missing tools | none |

Claude also picks these skills up on its own from natural questions such as
"how did Spring Campaign do last week" or "any new leads today".

## MCP tools

The server exposes only the tools the key has permission for.

- **Funnel**: `list_funnels`, `get_funnel`, `list_funnel_steps`, `create_funnel`,
  `update_funnel`, `set_funnel_active`, `create_funnel_step` (site step, or form
  step with `formId`), `set_funnel_step_visibility`, `set_funnel_step_form`,
  `set_form_step_completion`, `delete_funnel`
- **Page builder** (Funnel permission): `get_page_builder_reference`,
  `get_page_project`, `validate_page_project`, `save_page_project`, `publish_page`
- **Contact**: `list_contacts`, `list_leads`, `get_contact`, `get_contact_activity`,
  `list_contact_opt_ins`, `get_contact_pipeline_stats`, `get_contact_options`,
  `update_contact`, `create_contact_meeting`
- **Performance**: `list_funnel_performance`, `get_funnel_performance`,
  `compare_funnel_performance_with_previous_period`, `get_funnel_step_performance`,
  `get_funnel_conversion_stats`, `get_form_step_performance`, `compare_funnels`
- **Subscription**: `get_subscription`, `get_subscription_usage`
- **Form**: `list_forms`, `get_form`, `create_form`, `update_form`, `publish_form`,
  `delete_form`, `list_form_responses`, `get_form_performance`

Ids are GUIDs, dates are ISO 8601 with offset, lists are paged with `page`
and `itemsPerPage` (max 100).

## Development

The tool set is defined in the API repository under `API/MCP/Tools/`. When a
tool is added or renamed there, update the matching skill and the table above,
bump `version` in `.claude-plugin/plugin.json`, and run:

```bash
claude plugin validate .
```
