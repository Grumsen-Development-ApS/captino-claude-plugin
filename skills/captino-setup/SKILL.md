---
name: captino-setup
description: Set up or troubleshoot the Captino MCP connection in Claude Code - sign in to Captino with OAuth, choose which permissions to grant, re-authenticate after a revoke or expiry, check which tools the connection has, and fix 401, 403, "needs authentication", "needs additional permissions", missing tools or "Unable to connect" errors. Use when Captino tools are unavailable or failing, or when the user asks how to connect Claude Code to Captino.
---

# Captino setup

The plugin talks to `https://api.captino.io/mcp` over HTTP. The server uses
OAuth: Claude Code finds the Captino sign-in on its own, registers itself and
stores and refreshes the tokens, so there is nothing to configure.

The connection acts as the Captino user who approved it, in that user's
account. Access tokens last an hour and refresh on their own. The refresh token
lasts 30 days and is renewed each time it is used, so only a connection left
unused for 30 days has to sign in again.

## Connect

1. Enable the plugin in `/plugin`. Its server appears in `/mcp` as
   `plugin:captino:captino` and needs authentication.
2. Run `/mcp`, select `plugin:captino:captino` and choose **Authenticate**. A
   browser opens on captino.io.
3. Sign in to Captino. On the consent screen, tick the resources Claude Code
   may use (see below) and approve.
4. Back in Claude Code, the server shows as connected and its tools are listed.

`claude mcp login plugin:captino:captino` runs the same sign-in from a shell;
add `--no-browser` on a machine without a browser and paste the callback URL
when asked. If the browser shows a connection error after approving, paste the
full URL from its address bar into the prompt Claude Code shows.

## Permissions

The consent screen lists these resources. Write access implies read.

| Resource | Access levels | OAuth scopes | Unlocks |
| :-- | :-- | :-- | :-- |
| Funnel | read, write | `funnel:read`, `funnel:write` | `list_funnels`, `get_funnel`, `list_funnel_steps`, `get_page_builder_reference`, `get_page_project`, `validate_page_project`; write adds `create_funnel`, `update_funnel`, `set_funnel_active`, `create_funnel_step`, `set_funnel_step_visibility`, `set_funnel_step_form`, `set_form_step_completion`, `delete_funnel`, `save_page_project`, `publish_page` |
| Contact | read, write | `contact:read`, `contact:write` | `list_contacts`, `list_leads`, `get_contact`, `get_contact_activity`, `list_contact_opt_ins`, `get_contact_pipeline_stats`, `get_contact_options`; write adds `update_contact`, `create_contact_meeting` |
| Performance | read | `performance:read` | `list_funnel_performance`, `get_funnel_performance`, `compare_funnel_performance_with_previous_period`, `get_funnel_step_performance`, `get_funnel_conversion_stats`, `get_form_step_performance`, `compare_funnels` |
| Subscription | read | `subscription:read` | `get_subscription`, `get_subscription_usage` |
| Form | read, write | `form:read`, `form:write` | `list_forms`, `get_form`, `list_form_responses`, `get_form_performance`; write adds `create_form`, `update_form`, `publish_form`, `delete_form` |

The server only lists the tools the granted permissions allow, so a missing
tool almost always means a box that was not ticked, not a broken install.

Recommended grants for the skills in this plugin: Funnel write, Contact write,
Form write, Performance read, Subscription read. Funnel read is enough if the
user will not build pages or change funnels; Form is only needed for forms.

## Change permissions or disconnect

Connected apps are listed in Captino under **Settings → MCP**
(`/app/mcp-settings`), where the user can revoke one. Permissions cannot be
edited on an existing connection. To grant more or fewer, sign in again and
tick different boxes: in `/mcp`, select `plugin:captino:captino`, choose
**Clear authentication**, then **Authenticate**. Revoke the old entry under
Connected apps if it is still listed.

## Troubleshooting

Match the symptom, then do the fix. Do not retry a failing call against a
different URL or with a token from somewhere else.

| Symptom | Cause | Fix |
| :-- | :-- | :-- |
| No `plugin:captino:captino` in `/mcp`, no Captino tools | Plugin not enabled or not loaded | `/plugin` → enable Captino, then `/reload-plugins` |
| Server needs authentication, `401`, or `needs you to sign in again` | Not signed in yet, the app was revoked under Connected apps, or the refresh token expired after 30 days unused | `/mcp` → `plugin:captino:captino` → **Authenticate** (or **Re-authenticate**) |
| Token refresh fails and signing in again is refused | The Captino user who approved the app, or the account, is deactivated | Ask a Captino admin to reactivate it, or connect as an active user |
| Some tools missing, others work | That resource was not ticked on the consent screen | Sign in again and tick it (see Change permissions) |
| `403` on a tool call, or `needs additional permissions (scope: ...)` | Same as above, for a permission the call needs | Sign in again and grant the named scope |
| A hand-added `captino` server fails with `401` | It was added with an `Authorization` header, and Claude Code does not use OAuth for a server with a configured header | `claude mcp remove captino` and use the plugin's server |
| `Unable to connect` / `ConnectionRefused` | Network, VPN or proxy blocks `api.captino.io`, or the API is down | Check `curl -sI https://api.captino.io/mcp` (a 401 is the healthy answer without a sign-in); check the network, then `/mcp` → reconnect |
| `404 ... not found` on a specific id | The GUID belongs to another account or was deleted | Look the item up again with `list_funnels` or `list_contacts` |
| Contacts missing from lists, `hiddenContacts > 0` | Subscription contact limit reached | Not an auth issue. Upgrade the plan in Captino |

## Verify

After connecting, call `get_subscription` (or `list_funnels` if Subscription
is not granted). A successful answer confirms the sign-in, the account and the
permissions in one go. Report which resources were granted based on which
tools are listed.
