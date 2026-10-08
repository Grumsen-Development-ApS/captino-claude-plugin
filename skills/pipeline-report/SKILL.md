---
name: pipeline-report
description: Build a Captino account overview - pipeline counts and total deal value, opt-ins and traffic per funnel, best performing funnels with week-over-week change, and subscription usage against plan limits. Use when the user asks for a "weekly report", "status update", "how is the account doing", "summary of my funnels and leads", "monthly overview", or "am I close to my plan limits".
---

# Pipeline report

Assembles a one-page account report from several Captino MCP tools.

## Prerequisites

Uses `get_contact_pipeline_stats`, `list_funnels`, `list_funnel_performance`,
`compare_funnel_performance_with_previous_period`, `get_subscription` and
`get_subscription_usage`. Needs an MCP API key with read access to
**Contact**, **Funnel**, **Performance** and **Subscription**. Sections whose
tools are missing are skipped with a one-line note; do not fabricate them.
Run the `captino-setup` skill if nothing is available.

## Period

Default is the last 7 days ending today (a weekly report). "Monthly" means the
last 30 days or the calendar month the user names. State the period at the top.
Dates go to the tools as ISO 8601 with offset.

## Data collection

Run these calls; the first four are independent, so make them together.

1. `get_contact_pipeline_stats` → `contacts`, `leads`, `opportunities`,
   `customers`, `lost`, `totalValue`.
2. `list_funnels` with `orderBy: "optIns"`, `descending: true`,
   `itemsPerPage: 100` → per funnel `id`, `name`, `isActive`, `visitUrl`,
   all-time `uniqueVisitors`, `pageviews`, `optIns`, `formResponses`.
3. `list_funnel_performance` with `rankBy: "OptIns"` → all-time ranking.
4. `get_subscription` and `get_subscription_usage` → plan `state`
   (`None`, `Active`, `Trialing`, `Inactive`), trial and period dates,
   and for funnels, custom domains and contacts: `used`, `limit`,
   `remaining`, `isLimitReached`, plus `nextTier`.
5. For the top funnels by all-time opt-ins (at most 5, only active ones), call
   `compare_funnel_performance_with_previous_period` with the report period.
   Use `selectedGraphData.totalVisitors` / `totalOptIns` versus
   `pastGraphData` for week-over-week change.

## Report layout

Keep it to what fits on one screen. Use these sections, drop any that is empty.

**Headline** (2 sentences): the most important movement, for example opt-ins up
or down across the top funnels, and any limit that is reached.

**Pipeline**: one table, one row per stage, plus total deal value. Add win rate
= `customers / (customers + lost)` when both are non-zero.

**Funnels this period**: table of the top funnels with visits, opt-ins, opt-in
rate, and change vs the previous period. Flag inactive funnels or funnels with
`visitUrl: null` (nothing published) separately; they are not "low traffic",
they are off.

**Subscription**: plan name and state, used/limit for funnels, domains and
contacts. Call out `isLimitReached: true` explicitly: contacts over the limit
are hidden from lists, and new funnels or domains cannot be created. Mention
`nextTier` by name only when a limit is reached or within 10%. If `state` is
`Trialing`, give the trial end date.

**Suggested next steps** (max 3 bullets), each tied to a number above: review
N pending leads, look into the funnel whose opt-in rate fell, upgrade before
the contact limit hides more sign-ups. Point to the `funnel-dropoff` or `leads`
skills where they help.

Numbers go in tables, not prose. Label all-time counters as all-time so they
are not read as period numbers.
