---
name: funnel-performance
description: Analyze how a Captino funnel performs - visits, unique visits, opt-ins and opt-in rate over a period, the change against the previous period, the ranking of all funnels, or a head-to-head of two funnels. Use whenever the user asks "how is my funnel doing", "which funnel performs best", "did traffic or opt-ins go up", "compare funnel A with B", "funnel stats for last week", or any question about funnel numbers, even if they don't say "performance".
---

# Funnel performance

Answers questions about funnel traffic and opt-ins using the Captino MCP server.

## Prerequisites

Uses these Captino MCP tools: `list_funnels`, `get_funnel`, `list_funnel_performance`,
`get_funnel_performance`, `compare_funnel_performance_with_previous_period`,
`get_funnel_step_performance`, `compare_funnels`. They need an MCP API key with
**Funnel: read** and **Performance: read**. If the tools are missing or a call
returns 401/403, run the `captino-setup` skill instead of guessing. Never fabricate
numbers; if a tool is unavailable, say so.

## Workflow

### 1. Resolve the funnel

- Ids are GUIDs. Never guess one. If the user names a funnel, call `list_funnels`
  with `query` set to the name and pick the match. If several match, list them
  (name, domain, active) and ask which one.
- If the user says "my funnels" or "all funnels", skip to ranking (step 3a).
- Keep the `id`, `name`, `isActive` and `visitUrl` from the result. A funnel with
  `isActive: false` or `visitUrl: null` cannot receive traffic; mention this if its
  numbers are zero.

### 2. Resolve the period

Tools take `from` and `to` as ISO 8601 with offset (for example
`2026-09-01T00:00:00+02:00`). Work out the dates from today's date in the
user's timezone; the server converts to UTC.

| User says | Period |
| :-- | :-- |
| nothing / "recently" | last 30 days, ending today |
| "this week" | Monday of this week to today |
| "last week" | previous Monday to previous Sunday |
| "this month" / "last month" | first to last day of that month |
| "last 7 days", "since launch" | as stated; use `createdAt` from `list_funnels` for launch |

Always state the period you used in the answer.

### 3. Pick the tool for the question

| Question | Tool | Notes |
| :-- | :-- | :-- |
| a. Which funnel is best / rank my funnels | `list_funnel_performance` with `rankBy` | `OptIns` for "which converts best", `UniqueVisits` for reach, `AllVisits` (default) for raw traffic. All-time only, returns names and values, no ids. |
| b. How did funnel X do in a period | `get_funnel_performance` | Returns `dates`, `lines` (one series per metric), `totalVisitors`, `totalOptIns`, averages per day and week. |
| c. Is it up or down | `compare_funnel_performance_with_previous_period` | `selectedGraphData` is the requested period, `pastGraphData` the equally long period before it. |
| d. Traffic per step over time | `get_funnel_step_performance` | One line per step. For drop-off and conversion per step use the `funnel-dropoff` skill. |
| e. Funnel A vs funnel B | `compare_funnels` | All-time `allVisits` and opt-ins for `leftFunnel` and `rightFunnel`. |

For an overview of one funnel with no specific question, call b and c together
for the default period, plus `list_funnels` for the all-time counters.

### 4. Interpret

- Opt-in rate = `totalOptIns / totalVisitors`. Say "no traffic" rather than
  dividing by zero.
- Change vs previous period = `(current - past) / past`. If `past` is zero, report
  the absolute numbers instead of a percentage.
- Use the daily `lines` to spot spikes or dead days; name the date of the best
  and worst day when it is useful.
- All-time counters on `list_funnels` (`uniqueVisitors`, `pageviews`, `optIns`,
  `formResponses`) are not comparable to period numbers; don't mix them in one
  table without labelling.

### 5. Report

Lead with the answer in one or two sentences, then one compact table. Example:

> Spring Campaign got fewer visits than the week before but converted better.
>
> | Metric (1-7 Sep vs 25-31 Aug) | This period | Previous | Change |
> | :-- | --: | --: | --: |
> | Visits | 412 | 530 | -22% |
> | Opt-ins | 38 | 31 | +23% |
> | Opt-in rate | 9.2% | 5.8% | +3.4 pp |

End with at most two concrete observations (for example the day traffic dropped,
or that the funnel is inactive). Offer the `funnel-dropoff` skill when the opt-in
rate is the problem.
