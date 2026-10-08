---
name: leads
description: Review Captino leads and contacts - list new or recent leads, search a contact by name, email or phone, see which funnel a lead came from, their opt-ins, activity and meetings, and pipeline counts per status. Use when the user asks "any new leads", "who signed up this week", "show my leads", "find contact X", "what did this lead do", "where did this lead come from", or "how many leads, opportunities or customers do I have".
---

# Leads

Read-only review of leads and contacts from the Captino MCP server. To change
a lead (status, value, note, meeting) use the `lead-followup` skill.

## Prerequisites

Uses the Captino MCP tools `list_leads`, `list_contacts`, `get_contact`,
`get_contact_activity`, `list_contact_opt_ins`, `get_contact_pipeline_stats`
and `get_contact_options`. Needs an MCP API key with **Contact: read**. If the
tools are missing or fail with 401/403, run the `captino-setup` skill.

## Domain facts

- Pipeline `status` progression: `Contact` (captured from an opt-in) →
  `Lead` (submitted a form or was promoted) → `Opportunity` (deal under
  negotiation) → `Customer` (closed). `Lost` means they left the process.
  So "leads" in the narrow sense is `status = Lead`; "everyone who signed up"
  is all statuses. Ask or state which one you used when it matters.
- `qualityStatus`: `Pending` (not reviewed), `Good`, `NotRelevant`, `Spam`.
- Lists are paged: `page` (1-based) and `itemsPerPage` (max 100). Responses
  include `hasNextPage` and `totalPages`.
- Contacts beyond the subscription limit are hidden. When a response has
  `hiddenContacts > 0`, always tell the user how many are hidden and that
  upgrading the plan reveals them.
- There is no server-side date filter. For "this week" or "since Monday",
  order by `createdAt` descending and stop paging once `createdAt` is older
  than the cutoff.

## Workflow

### New or recent leads

1. Call `list_leads` (or `list_contacts` with `status` if the user means
   another stage) with `itemsPerPage: 50`. `list_leads` is newest first.
   For `list_contacts` pass `orderBy: "createdAt"`, `descending: true`.
2. Filter by the cutoff date locally. Page further only while the last item is
   still inside the period.
3. Present a table: name, email, status, quality, created (date only),
   deal value if non-zero. Group by day if there are more than ~15.
4. If the user wants the source funnel for each lead, call
   `list_contact_opt_ins` per lead (this is one call per contact, so do it
   for at most ~10 leads unless asked) and add a "Source funnel" column.

### Find a contact

1. `list_contacts` with `query` set to the name, email or phone fragment.
2. If exactly one result, call `get_contact` for full details. If several,
   show them and ask which one.

### What did this lead do

1. Resolve the contact as above.
2. `get_contact_activity` returns `funnelActivities` (per funnel: visits,
   opt-ins, form submissions with timestamps), `meetings` and `lastActivity`.
3. `list_contact_opt_ins` returns each form sign-up with `funnelName`,
   `funnelStepId`, `acceptedTerms`, `timeZone`, `createdAt`.
4. Summarise as a short timeline: first seen, funnels touched, forms submitted,
   meetings held or scheduled, last activity. Note if `acceptedTerms` is false.

### Pipeline counts

Call `get_contact_pipeline_stats`. It returns `contacts`, `leads`,
`opportunities`, `customers`, `lost` and `totalValue`. Present as one row per
stage. Compute a simple win rate = `customers / (customers + lost)` only when
both are non-zero.

## Reporting

Lead with the answer ("You got 12 new leads this week, 9 from Spring Campaign"),
then the table. Keep personal data to what the user needs; don't dump notes or
phone numbers unless asked. Suggest `lead-followup` when the user's next step is
to qualify or move a lead.
