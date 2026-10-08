---
name: funnel-dropoff
description: Find where visitors leave a Captino funnel - visits and conversion rate per step, the biggest step-to-step drop, and per-question response counts of form steps. Use when the user asks "where do people drop off", "which step loses visitors", "conversion per step", "why doesn't my form convert", "which question do people abandon", or wants to improve a funnel's opt-in rate.
---

# Funnel drop-off

Diagnoses where a funnel loses visitors, step by step and, for form steps,
question by question.

## Prerequisites

Uses the Captino MCP tools `list_funnels`, `list_funnel_steps`,
`get_funnel_conversion_stats` and `get_form_step_performance`. Needs an MCP API
key with **Funnel: read** and **Performance: read**. If they are missing or fail
with 401/403, run the `captino-setup` skill. Never invent numbers.

## Workflow

### 1. Resolve the funnel and its steps

1. Find the funnel id with `list_funnels` (`query` = the name). Ask if the name
   is ambiguous.
2. Call `list_funnel_steps` with the funnel id. Note each step's `order`, `name`,
   `type` (`Site` or `Form`), `isPublic`, `hasDraft` and `hasFormDraft`.
   - Unpublished steps (`isPublic: false`) get no traffic; they are not
     drop-off, they are gaps in the funnel. Say so.
   - `hasDraft` / `hasFormDraft` means the live page differs from what the user
     sees in the editor. Mention it when the user is puzzled by the numbers.

### 2. Get per-step conversion

Call `get_funnel_conversion_stats` with the funnel id. Pass `from` and `to`
(ISO 8601 with offset) when the user names a period; omit both for all-time.

Each item has `order`, `funnelStepName`, `visits` and `conversionRate`.
`conversionRate` is the percentage of the **first step's** visits that reached
this step, not the rate from the previous step. Compute:

- step-to-step retention = `visits[n] / visits[n-1]`
- step drop = `visits[n-1] - visits[n]` (absolute) and `1 - retention` (relative)

The biggest problem is usually the largest **relative** drop between two
consecutive published steps, but show the absolute numbers too because a 50%
drop on 10 visitors matters less than a 20% drop on 5,000.

### 3. Drill into form steps

For every step with `type: Form` that sits at or right after the biggest drop,
call `get_form_step_performance` with the step id and a period (`from` and `to`
are required here; use the same period as step 2, or the last 30 days for
all-time questions).

The result lists `questions` in order, each with `responses`, plus total
`submissions`. Responses fall as visitors abandon the form, so the largest drop
between consecutive questions is the question that costs the most submissions.
`submissions` versus the first question's responses is the overall form
completion rate.

### 4. Report

Lead with the single biggest leak in one sentence, then one table of the steps:

> Most visitors leave between the landing page and the quiz: only 31% continue.
>
> | # | Step | Visits | From first step | From previous step |
> | --: | :-- | --: | --: | --: |
> | 1 | Landing | 1,240 | 100% | - |
> | 2 | Quiz (form) | 385 | 31% | 31% |
> | 3 | Thank you | 290 | 23% | 75% |

If you drilled into a form, add a second small table of questions and their
response counts and name the question where abandonment peaks.

Close with two or three concrete suggestions tied to the numbers: shorten the
form before question N, move the value proposition above the fold on step 1,
publish the missing step, and so on. Keep them specific to what the data shows.
