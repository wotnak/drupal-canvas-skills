---
name: canvas-page-templates
description: >-
  Create, edit, assign, or validate Drupal Canvas page templates. Use for shared
  headers, navigation, footers, and other full-page layouts.
---

# Canvas page templates

Put shared headers, navigation, and footers in the page template's component
tree. Canvas inserts the rendered page content at the `marker.page_content`
element.

## Author a page template

Put the JSON file directly in `pageTemplatesDir` from `canvas.config.json`,
which defaults to `page-templates/`. The filename `<id>.json` gives the variant
its machine name. Use lowercase letters, digits, and underscores
(`^[a-z0-9_]+$`). Keep the filename when editing because assignments refer to
its machine name.

- `label` is a required, non-empty display name.
- `elements` is a required object keyed by element ID. Each element has a
  component `type` and may have literal `props` and `slots`. Top-level elements
  render in insertion order. Slot arrays give the order of their children and
  reference element IDs in the same file.
- Include exactly one Canvas-provided `marker.page_content` element without
  props or slots. Place it at the top level or in a component slot, wherever the
  main content should appear.
- `description` is optional. `status` and `default` are optional booleans and
  default to `true` and `false`, respectively. At most one local file may set
  `"default": true`, and that template must be enabled. Pushing that file makes
  it the site default.

For example, `page-templates/standard.json` on a project with the
`js.site_header` and `js.site_footer` components can contain:

```json
{
  "label": "Standard",
  "elements": {
    "header": { "type": "js.site_header" },
    "main": { "type": "marker.page_content" },
    "footer": { "type": "js.site_footer" }
  }
}
```

## Assign a page template

Set `"pageVariant": "standard"` at the top level of a page spec or a `full`
view-mode content template spec to select `standard.json`. Omit `pageVariant` to
inherit the site default. Keep the page's own content in its `elements`; the
marker belongs to the page template. A teaser or other non-full view mode
renders its own content without page template chrome.

In Canvas, manage templates under **Templates → Page templates**. Use **Page
data → Page template** on a Canvas page or the **Page template** submenu of a
full content template to change its selection. Use the template's **Set as
default** action for a site-wide change.

## Validate, preview, and sync

Run `npx canvas validate` from the project root. In Workbench, open the page
template and a page that uses it. Workbench resolves `pageVariant` against local
specs or uses the local template marked `"default": true` when the page has no
explicit selection. A site-managed template must have a local spec to appear in
Workbench. Check both an inheriting page and an explicitly assigned page when
changing the default.

`canvas pull`, `canvas push`, and `canvas reconcile-media` include page
templates by default. A custom OAuth scope list needs `canvas:page_variant`.
Pull skips templates that contain components other than code components or the
page content marker, including theme-backed templates.

Review the CLI's proposed operations before an authorized push. Page template
sync can delete remote templates missing locally, including ones skipped on
pull. Use `sync.pageTemplates: false` in `canvas.config.json` or
`--no-page-templates` for a push that should leave page templates managed in
Drupal. Pages can still reference existing remote template IDs in that push.

In headless projects, use components registered for the frontend renderer and
verify the composed page through the app's Canvas route. Keep Canvas-managed
headers and footers in the template so each appears once on that route. The
`canvas-headless` skill covers renderer setup.
