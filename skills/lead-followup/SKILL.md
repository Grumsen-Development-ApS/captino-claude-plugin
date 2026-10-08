---
name: lead-followup
description: Act on a Captino lead or contact - set its quality (Good, NotRelevant, Spam), move it through the pipeline (Lead to Opportunity, Customer or Lost), set the deal value, add a note, or schedule a meeting. Use when the user says "mark X as opportunity", "this lead is spam", "we closed the deal with X", "set the value to", "add a note to", "book a meeting with", or otherwise wants to update a contact.
---

# Lead follow-up

Writes to contacts through the Captino MCP server. Every change here is visible
to the whole Captino account, so be precise about which contact you change and
what you change.

## Prerequisites

Uses the Captino MCP tools `list_contacts`, `get_contact`, `get_contact_options`,
`update_contact` and `create_contact_meeting`. Needs an MCP API key with
**Contact: write** (write implies read). If `update_contact` or
`create_contact_meeting` is not listed, the key is read-only; tell the user and
point them to the `captino-setup` skill instead of trying workarounds.

## Domain facts

- `status` values: `Contact`, `Lead`, `Opportunity`, `Customer`, `Lost`.
- `qualityStatus` values: `Pending`, `Good`, `NotRelevant`, `Spam`.
- `value` is the monetary deal value (plain decimal, account currency).
- `update_contact` changes only the arguments you pass. Omitted arguments keep
  their value. **Exception:** `note` replaces the whole note.
- `create_contact_meeting` takes `scheduledAt` as ISO 8601 with offset, optional
  `title` and `note`, and updates the contact's `meetingDate` automatically.

## Workflow

### 1. Resolve the contact

Call `list_contacts` with `query` set to the name, email or phone the user gave.
Exactly one match: continue. Several matches: show them (name, email, status,
created) and ask which one. None: say so; do not create contacts, there is no
tool for it.

### 2. Show the current state before changing it

Call `get_contact` and keep `status`, `qualityStatus`, `value`, `note` and
`meetingDate`. Sanity-check the request against it:

- Moving backwards in the pipeline (for example `Customer` → `Lead`) or to
  `Lost` from `Customer` is unusual. Do it if asked, but say what the current
  status is.
- Setting `Spam` or `NotRelevant` on a contact with a non-zero deal value or a
  scheduled meeting deserves a one-line heads-up.
- If the user asks to "add" to the note, fetch the existing note and send the
  existing text plus the addition; sending only the addition would erase it.

### 3. Apply the change

- Status, quality, value, name, email, phone, note: one `update_contact` call
  with only the fields that change.
- Meeting: `create_contact_meeting`. If the user gave no time, ask for it; do
  not pick one. Use the user's timezone offset. Put the agenda in `note` and a
  short label in `title`.
- Closing a deal usually means two things: `status: Customer` and `value`. If
  the user gives a value, send both in the same call.
- Several contacts at once (for example "mark all leads from yesterday as
  Good"): list them first, show the list, and get an explicit go-ahead before
  looping. There is no bulk tool; make one call per contact and report any that
  failed.

### 4. Confirm

Report before → after for the fields that changed, in one line per contact:

> Anna Berg: Lead → Opportunity, value 0 → 12,000. Note kept.
> Meeting scheduled 12 Sep 2026 10:00 (+02:00), "Intro call".

If a call failed (404 not found, 403 no write permission, validation error),
quote the error and stop; do not retry with guessed data.
