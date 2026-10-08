---
name: page-builder
description: Build, edit and publish Captino funnel step pages (landing pages, opt-in pages, thank-you pages, popups) from a brief, using the page builder MCP tools and GrapesJS project data made of Captino components. Use when the user asks to "build a landing page", "create a page for my funnel", "add an opt-in page", "change the headline on my page", "add a popup", "publish the page", or wants a website or funnel page written or changed without opening the visual builder.
---

# Page builder

Writes or changes the page of a funnel step as GrapesJS project data built from
Captino components, checks it with the compiler, saves it as a draft and, when
the user wants, publishes it. The result is the same as a page made in the
visual builder, so the user can keep editing it there afterwards.

## Prerequisites

Uses the Captino MCP tools `get_page_builder_reference`, `get_page_project`,
`validate_page_project`, `save_page_project`, `publish_page`,
`create_funnel_step`, `list_funnels`, `get_funnel` and `list_funnel_steps`.
Needs an MCP API key with **Funnel: write** (read is enough for the reference,
reading a project and validating). If the tools are missing or fail with
401/403, run the `captino-setup` skill.

Only **site steps created with the page builder** can be built this way. Form
steps and legacy template steps are rejected with "not a page builder step";
build and attach forms with the `form-builder` skill.

## Workflow

### 1. Understand the brief

Before writing anything, know: the goal of the page (collect leads, sell,
inform, thank), the audience, the single call to action, the tone, and any
brand colors, fonts, images or copy the user already has. Ask only for what
you cannot infer; one short question is better than a form. Use the user's own
words for the copy when they gave some, and write concise, concrete copy when
they did not. Never invent testimonials, numbers or claims about the user's
product.

### 2. Resolve the funnel and the step

1. Find the funnel with `list_funnels` (`query` = the name), then `get_funnel`
   for its `steps`, `gdprCompliant` and `visitUrl`. Ask if the name is ambiguous.
2. Pick the step:
   - Editing an existing page: the step the user named. Check `type: Site`,
     `editorType: GrapesJs`, and note `isPublic` and `hasDraft`.
   - New page: `create_funnel_step` with the funnel id and a name such as
     "Landing page". It is appended to the funnel, unpublished, with a path
     derived from the name. A new funnel needs `create_funnel` first (name and
     subdomain).
3. Note the `path` of every step. The opt-in form's redirect target is a step
   path, and buttons that link between steps use `/{path}`.

### 3. Read the reference once

Call `get_page_builder_reference` once per conversation and keep it. It is the
source of truth: component types with their properties (`traits`), allowed
values (`options`), defaults, the region rules and a small valid example. The
summary below is for orientation; when they disagree, the reference wins.

### 4. Author the project data

The project is one JSON object:

```json
{
  "pages": [{ "frames": [{ "component": { "type": "wrapper", "components": [ ...regions or sections... ] } }] }],
  "styles": [],
  "assets": []
}
```

Rules:

- Exactly one page, one frame, a `wrapper` root.
- A component is `{ "type": "captino-…", "components": [...] | "text", "attributes": {...}, "cap-…": value }`.
  Traits with `changeProp: true` are top-level `"cap-…"` properties; the rest
  (`href`, `id`, `title`) go in `attributes`. Text and button labels are a
  string in `components`; inline HTML is limited to `b`, `i`, `br`, `span`.
- Structure: `wrapper` › `captino-region` (`cap-region-kind`: `header`,
  `template`, `footer`, `popups`) › `captino-section` (or `captino-modal` in
  `popups`) › `captino-stack` / `captino-columns` › content. Regions may be
  omitted; stray sections land in the template region.
- Use Captino components only. Never raw HTML elements; `captino-custom-code`
  is the last resort for embeds.
- Give components you may want to style or reference an `attributes.id`.
  Custom CSS goes in `styles` as `{ "selectors": ["#id"], "style": { ... } }`,
  but prefer component properties.
- Images and videos are absolute URLs the user provided or that already exist
  in Captino. Do not make up image URLs; leave images out or ask.
- Values of select properties must be one of the listed options, as strings.

Component summary (properties in the reference are authoritative):

| Component | Use | Key properties |
| :-- | :-- | :-- |
| `wrapper` | Page theme | `cap-accent-primary`, `cap-accent-secondary`, `cap-bg`, `cap-font-family`, `cap-heading-font-family`, `cap-radius-scale` (`default`, `sharp`, `rounded`, `pill`), `cap-section-spacing`, `cap-button-style` |
| `captino-section` | Full-width band | `cap-max-width` (`none`…`1200px`), `cap-padding-y`, `cap-padding-x`, `cap-bg`, `cap-bg-image` |
| `captino-stack` | Flex column or row | `cap-direction` (`column`, `row`), `cap-align`, `cap-justify`, `cap-gap`, `cap-padding`, `cap-wrap`, `cap-collapse` |
| `captino-columns` › `captino-column` | Side-by-side layout | `cap-gap`, `cap-collapse` (`never`, `tablet`, `mobile`); children must be `captino-column` |
| `captino-text` | Any text | `cap-format` (`title`, `subtitle`, `body`, `small`, `quote`), `cap-align`, `tagName` (`h1`…`h6`, `p`) |
| `captino-button` | CTA link or modal opener | `attributes.href`, `cap-variant` (`primary`, `secondary`, `outline`, `ghost`), `cap-corners`, `cap-action` (`link`, `open-modal`), `cap-modal-target`, `cap-link-new-tab` |
| `captino-optin-form` | Lead capture, wired to Captino | `cap-show-name/email/phone/company`, labels and placeholders, `cap-submit-label`, `cap-after-submit` (`message`, `close-modal`, `redirect-step`, `redirect-url`), `cap-redirect-step` (a step path), `cap-redirect-url` |
| `captino-image` / `captino-video` | Media | `cap-src` + `cap-alt` / `cap-url` (YouTube or Vimeo), `cap-fit`, `cap-corners`, `cap-height`, `cap-ratio` |
| `captino-bullet-list` › `captino-bullet-item` | Benefit lists | `cap-marker` (`disc`, `check`, `arrow`) |
| `captino-modal` › `captino-modal-content` | Popup in the `popups` region | `cap-modal-id`, `cap-trigger` (`button`, `load`), `cap-delay`, `cap-size` |
| `captino-input`, `captino-checkbox`, `captino-map`, `captino-custom-code` | Standalone field, checkbox, Google Map, embed | see reference |

Patterns that work:

- **Hero**: section (`cap-max-width: 960px`) › stack (`cap-align: center`,
  `cap-gap: 24px`) › `captino-text` title as `h1` centered, `captino-text`
  subtitle, `captino-button` or `captino-optin-form`.
- **Benefits**: section › `captino-columns` with three `captino-column`s, each
  holding a `captino-text` subtitle and a body text, or one
  `captino-bullet-list` with `cap-marker: check`.
- **Opt-in page**: hero copy on the left column, `captino-optin-form` on the
  right; `cap-after-submit: redirect-step` with `cap-redirect-step` set to the
  thank-you step's path. Show only the fields the user needs; every extra field
  lowers conversion.
- **Popup**: `captino-modal` (`cap-modal-id: "offer"`) in the `popups` region
  holding a `captino-modal-content` with the form; a `captino-button` with
  `cap-action: open-modal` and `cap-modal-target: "offer"` opens it.
- **Footer**: `footer` region › section › stack row with small texts. The
  funnel's GDPR links are added by Captino when `gdprCompliant` is on; do not
  hand-write a cookie banner.

Set the theme on the `wrapper` once (accent color, fonts, radius) instead of
coloring individual components; the components follow the theme.

Minimal example:

```json
{
  "pages": [{ "frames": [{ "component": {
    "type": "wrapper",
    "cap-accent-primary": "#7581F0",
    "components": [{
      "type": "captino-region", "cap-region-kind": "template",
      "components": [{
        "type": "captino-section", "cap-max-width": "960px",
        "components": [{
          "type": "captino-stack", "cap-align": "center", "cap-gap": "24px",
          "components": [
            { "type": "captino-text", "tagName": "h1", "cap-format": "title", "cap-align": "center", "components": "Grow your audience" },
            { "type": "captino-text", "cap-align": "center", "components": "Join 2,000+ marketers who get our weekly playbook." },
            { "type": "captino-optin-form", "cap-show-name": false, "cap-submit-label": "Send me the playbook", "cap-after-submit": "redirect-step", "cap-redirect-step": "thank-you" }
          ]
        }]
      }]
    }]
  } }] }],
  "styles": [],
  "assets": []
}
```

### 5. Validate, then save

1. `validate_page_project` with the funnel id, step id and the project. It
   returns `isValid` plus `errors`, each naming the JSON path of the offending
   node, and `warnings`. Fix every error and validate again; do not save an
   invalid project. Pass `includeHtml: true` only when you need to inspect the
   output.
2. `save_page_project` with `publish: false` stores the compiled project as
   the step's draft. The response carries the step (`hasDraft: true`),
   `warnings` and `htmlLength`.
3. Publishing makes the page live and the step public. Only do it when the user
   asked for it or confirms: `save_page_project` with `publish: true`, or
   `publish_page` for a draft saved earlier. Publishing needs an active
   subscription; report the error plainly if it fails. The draft is kept.

### 6. Editing an existing page

1. `get_page_project` returns `source` (`draft`, `published` or `empty`) and
   the `projectData`. When `source` is `draft`, tell the user there are
   unpublished changes that the edit will build on.
2. Change only what the user asked for. Keep existing `attributes.id`,
   `classes`, `styles` and component order; the visual builder stored them.
3. Validate and save as in step 5. Saving always writes the draft; the live
   page changes only on publish.

### 7. Report

State what was built or changed and where it is, in a few lines: the funnel and
step names, the step path, whether it is a draft or live, and the page URL
(`visitUrl` from `get_funnel` plus `/{path}`) when it is live. Mention warnings
from the compiler only when they need action. Offer the next step once: publish
the draft, add a thank-you page, or wire the form's redirect.

## Pitfalls

- `pages` must contain exactly one page; a funnel step is one page.
- Unknown `type` values and select values outside the options are errors, not
  warnings. Read them from the reference instead of guessing.
- `cap-redirect-step` takes the target step's **path**, not its id.
- A `captino-columns` may only contain `captino-column`; a `captino-modal`
  only lives in the `popups` region; the `header`, `template` and `footer`
  regions only accept `captino-section`.
- `attributes.class` is accepted but `classes` is what the builder uses; do not
  add framework CSS classes, they are not on the published page.
- Do not paste whole HTML documents into `captino-custom-code`; build the page
  from components so the user can keep editing it visually.
