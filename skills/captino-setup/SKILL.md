---
name: captino-setup
description: Set up or troubleshoot the Captino MCP connection in Claude Code - create an MCP API key, store it in the plugin, check which tools and permissions the key has, and fix 401, 403, "Invalid MCP API key", missing tools or "Unable to connect" errors. Use when Captino tools are unavailable or failing, or when the user asks how to connect Claude Code to Captino.
---

# Captino setup

The plugin talks to `https://api.captino.io/mcp` over HTTP with a bearer token.
The token is a Captino MCP API key.

## Get an API key

1. Open the Captino app and go to **Settings → MCP** (route `/app/mcp-settings`).
2. Create a key and grant the resources the user needs. Keys start with
   `cap_mcp_` and are shown once; copy it immediately.

| Resource | Access levels | Unlocks |
| :-- | :-- | :-- |
| Funnel | read, write | `list_funnels`, `get_funnel`, `list_funnel_steps`, `get_page_builder_reference`, `get_page_project`, `validate_page_project`; write adds `create_funnel`, `update_funnel`, `set_funnel_active`, `create_funnel_step`, `set_funnel_step_visibility`, `set_funnel_step_form`, `set_form_step_completion`, `delete_funnel`, `save_page_project`, `publish_page` |
| Contact | read, write | `list_contacts`, `list_leads`, `get_contact`, `get_contact_activity`, `list_contact_opt_ins`, `get_contact_pipeline_stats`, `get_contact_options`; write adds `update_contact`, `create_contact_meeting` |
| Performance | read | `list_funnel_performance`, `get_funnel_performance`, `compare_funnel_performance_with_previous_period`, `get_funnel_step_performance`, `get_funnel_conversion_stats`, `get_form_step_performance`, `compare_funnels` |
| Subscription | read | `get_subscription`, `get_subscription_usage` |
| Form | read, write | `list_forms`, `get_form`, `list_form_responses`, `get_form_performance`; write adds `create_form`, `update_form`, `publish_form`, `delete_form` |

Write access implies read. The server only lists the tools the key is allowed
to call, so a missing tool almost always means a missing permission, not a
broken install.

Recommended grants for the skills in this plugin: Funnel write, Contact write,
Form write, Performance read, Subscription read. Funnel read is enough if the
user will not build pages or change funnels; Form is only needed for forms.

## Store the key in the plugin

The plugin asks for the key when it is enabled and stores it in secure storage
as the `api_key` option. To enter or change it later, run `/plugin`, open the
Captino plugin and edit its configuration, then run `/reload-plugins` or
restart Claude Code. Check the connection with `/mcp`.

When testing the plugin from a local checkout with `--plugin-dir`, the same
prompt appears on first use.

## Troubleshooting

Match the symptom, then do the fix. Do not retry a failing call with a guessed
key or a different URL.

| Symptom | Cause | Fix |
| :-- | :-- | :-- |
| No `captino` server in `/mcp`, no Captino tools | Plugin not enabled or not loaded | `/plugin` → enable Captino, then `/reload-plugins` |
| `401` with `Invalid MCP API key` | Key mistyped, revoked, or missing the `cap_mcp_` prefix | Create a new key in Settings → MCP and re-enter it in `/plugin` |
| `401` with `User is deactivated` or `Tenant is deactivated` | The user who created the key, or the account, is disabled | Ask a Captino admin; create the key as an active user |
| Some tools missing, others work | Key lacks that resource or access level | Edit the key's permissions in Captino (or create a new key) |
| `403` on a tool call | Same as above, for a tool that was listed before the key changed | `/reload-plugins` after fixing permissions |
| `Unable to connect` / `ConnectionRefused` | Network, VPN or proxy blocks `api.captino.io`, or the API is down | Check `curl -sI https://api.captino.io/mcp` (a 401 is the healthy answer without a key); check the network, then `/mcp` → reconnect |
| `404 ... not found` on a specific id | The GUID belongs to another account or was deleted | Look the item up again with `list_funnels` or `list_contacts` |
| Contacts missing from lists, `hiddenContacts > 0` | Subscription contact limit reached | Not an auth issue. Upgrade the plan in Captino |

## Verify

After setup, call `get_subscription` (or `list_funnels` if Subscription is not
granted). A successful answer confirms the key, the tenant and the permissions
in one go. Report which resources the key has based on which tools are listed.
