---
name: form-builder
description: Build, edit and publish Captino forms (multi-question lead forms, qualification forms, surveys, applications, quizzes) from a brief and put them into a funnel as a form step, using the form MCP tools. Use when the user asks to "create a form", "build a lead form", "add a survey to my funnel", "change the questions on my form", "add a question", "restyle the form", "publish the form", or wants a form written or changed without opening the form builder.
---

# Form builder

Writes or changes a standalone Captino form as a form configuration (questions,
intro, completion, labels, style), saves it as a draft, publishes it when the
user wants, and shows it in a funnel through a form step. The result is the
same as a form made in the form builder, so the user can keep editing it there.

A form shows one question per screen with Back/Next navigation and lives on its
own funnel step. For a simple name-and-email opt-in inside a landing page, use
the `page-builder` skill and its `captino-optin-form` component instead.

## Prerequisites

Uses the Captino MCP tools `list_forms`, `get_form`, `create_form`,
`update_form`, `publish_form`, `list_funnels`, `get_funnel`,
`list_funnel_steps`, `create_funnel_step`, `set_funnel_step_form` and
`set_form_step_completion`. Needs an
MCP API key with **Form: write** to author forms and **Funnel: write** to add
them to a funnel. If the tools are missing or fail with 401/403, run the
`captino-setup` skill.

## Workflow

### 1. Understand the brief

Before writing anything, know: what the form is for (capture leads, qualify
them, book a call, collect feedback), the audience, the questions the user
needs answered, the language of the form, where respondents go after
submitting, and any brand colors, font or logo. Ask only for what you cannot
infer. Keep it short: every extra question lowers completion. Use the user's
own wording for questions when they gave some. Never invent claims about the
user's product.

### 2. Find or create the form

- Editing: `list_forms` with `query` = the name, then `get_form`. Note
  `hasDraft` and `linkedFunnels`. A form can be used by several funnels, and
  publishing changes it in all of them; tell the user when `linkedFunnels` has
  more than the one they mean.
- New form: `create_form` with a `name` and the full `config`. The config is
  saved as a draft; nothing is live yet.

### 3. Author the configuration

The configuration is one JSON object:

```json
{
  "intro": { "enabled": true, "title": "…", "description": "…" },
  "questions": [ { "id": "…", "question": "…", "type": "…", "required": true, "options": [], "placeholder": "" } ],
  "completion": { "message": "…", "redirectUrl": "" },
  "labels": { "start": "Start", "back": "Back", "next": "Next", "submit": "Submit", "acceptTerms": "I accept the terms and conditions", "continue": "Continue" },
  "style": { "accentColor": "#7581F0", "accentForegroundColor": "#ffffff", "foregroundColor": "#111827", "backgroundColor": "#ffffff", "font": "Inter", "corners": "Default", "brandAssetUrl": "", "brandAssetSize": "Small" }
}
```

Question types (exact spelling):

| Type | Use | Notes |
| :-- | :-- | :-- |
| `Name` | Respondent's name | Stored as the contact's name |
| `Email` | Email address | Creates or updates the contact |
| `Mobile` | Phone number | Creates or updates the contact |
| `ShortText` / `LongText` | Free text | `placeholder` optional |
| `SingleChoice` / `MultipleChoice` | Pick one / pick several | `options` is a list of strings |
| `Scale` | Rating | `scaleMin` and `scaleMax` within 0 to 10, min below max; defaults 0 and 10 |

Rules:

- Each question needs an `id` that is unique within the form. Use short,
  readable slugs (`email`, `company-size`). Per-question statistics are keyed
  by the id, so never change the id of an existing question.
- A response only becomes a lead when it has an `Email` or `Mobile` answer.
  For a lead form, include a required `Email` (or `Mobile`) question and a
  `Name` question.
- Put the easy questions first and contact details near the end, unless the
  user wants the lead captured before the qualifying questions.
- `options` only apply to choice questions and `placeholder` only to text
  questions (`Name`, `Email`, `Mobile`, `ShortText`, `LongText`).
- `intro` is an optional welcome screen with a Start button. Turn it on when
  the form needs context ("Takes 2 minutes"); leave it off for short forms.
- Write every label in the form's language. The defaults are English.
  `acceptTerms` is shown next to Submit only when the funnel has GDPR turned
  on.
- `style.corners` is one of `Default`, `Sharp`, `Rounded`, `Pill`;
  `brandAssetSize` one of `Small`, `Medium`, `Large`. `brandAssetUrl` is an
  absolute image URL the user provided; do not make one up.

Completion (what happens after submit):

- `message` only: the message is shown after submit.
- `redirectUrl` only (set `message` to `""`): the respondent goes straight to
  that URL.
- Both: the message is shown with a Continue button that opens the URL.

The form's `completion` applies in every funnel that uses the form. Set a
redirect here only for a target outside Captino or a form used in one place;
a thank-you page in the funnel belongs on the step (step 5), which points at
the step itself instead of a typed-in URL.

Minimal lead form:

```json
{
  "intro": { "enabled": false, "title": "", "description": "" },
  "questions": [
    { "id": "goal", "question": "What do you want help with?", "type": "SingleChoice", "options": ["More leads", "Better conversion", "Something else"], "required": true },
    { "id": "name", "question": "What is your name?", "type": "Name", "required": true },
    { "id": "email", "question": "Where should we send your plan?", "type": "Email", "required": true }
  ],
  "completion": { "message": "Thanks! Your plan is on its way.", "redirectUrl": "" },
  "labels": { "start": "Start", "back": "Back", "next": "Next", "submit": "Send me the plan", "acceptTerms": "I accept the terms and conditions", "continue": "Continue" },
  "style": { "accentColor": "#7581F0", "accentForegroundColor": "#ffffff", "foregroundColor": "#111827", "backgroundColor": "#ffffff", "font": "Inter", "corners": "Rounded", "brandAssetUrl": "", "brandAssetSize": "Small" }
}
```

### 4. Save and publish

1. `create_form` (new) or `update_form` (existing) saves the configuration as
   the draft. The live form does not change.
2. Publishing makes the draft live and re-renders every funnel step that uses
   the form, in every funnel. Only do it when the user asked for it or
   confirms: `publish_form` with the form id.

### 5. Put it in a funnel

1. Find the funnel with `list_funnels`, then `get_funnel` for its steps.
2. New form step: `create_funnel_step` with the funnel id, a name such as
   "Application" and `formId`. The step is appended with a path derived from
   the name. If the form is already published, the step goes live at once, so
   confirm first; otherwise it goes live on the first `publish_form`.
3. Existing form step (`type: Form`): `set_funnel_step_form` swaps the form it
   shows. Site steps cannot show a form.
4. Thank-you page: `set_form_step_completion` with the funnel id, the form
   step id and `redirectStepId` = the id of a site step in the same funnel. It
   applies to this step only and wins over the form's own completion. The
   step's URL is looked up each time the form step's page is rendered (on this
   call and on every `publish_form`), so it picks up a renamed path or a new
   domain on the next render. Publish the thank-you page too, or respondents
   land on a 404.
   - `message` replaces the form's message for this step; `redirectUrl` (an
     absolute URL) is for targets outside the funnel. Pass `redirectStepId` or
     `redirectUrl`, not both.
   - Empty fields keep the form's value, so a step cannot blank the message.
     For a redirect without a message, the form's own `completion.message`
     must be `""`; otherwise the message shows with a Continue button.
   - Each call replaces the step's previous override; resend `message` to
     keep it. A call with no fields removes the override.
   - Form steps in `get_funnel` and `list_funnel_steps` show their current
     `completionOverride`. Check it before changing the form's completion:
     an override wins, so the change would not show on that step.
5. To send visitors to the form, link to the step's path from a page (a
   `captino-button` with `href` `/{path}`, see `page-builder`), or share the
   step URL.

### 6. Editing an existing form

1. Start from `draftConfig` when `hasDraft` is true, otherwise from `config`.
   Tell the user when there are unpublished changes that the edit builds on.
2. Change only what the user asked for and send the whole configuration back
   with `update_form`. It replaces the draft: any section you leave out falls
   back to its defaults.
3. Keep existing question ids and order unless the user asked to change them.
4. Publish as in step 4.

### 7. Report

State what was built or changed in a few lines: the form name, the questions
in order with their types, whether it is a draft or live, the funnel step and
its URL when it is in a funnel (`visitUrl` plus `/{path}`), where
respondents go after submitting, and the other funnels a publish affected. Offer the next step once: publish the draft, add
it to a funnel, or add a thank-you page.

## Pitfalls

- `update_form` and `create_form` take the whole configuration, not a patch.
  Always send every section.
- Question `type`, `corners` and `brandAssetSize` are case-sensitive strings.
- The default completion `message` is not empty. Set it to `""` explicitly
  when the form should redirect straight away.
- Publishing a shared form changes it in every linked funnel. Check
  `linkedFunnels` first; for a funnel-specific variant, create a new form and
  `set_funnel_step_form` on that funnel's step.
- `delete_form` fails while any step uses the form and cannot be undone. Only
  call it when the user asks, after confirming.
- Changing the form's `completion` has no visible effect on a step with a
  `completionOverride`; change the step with `set_form_step_completion`
  instead.
- There is no tool for detaching a form from a step; send the user to the
  Captino app for that.
