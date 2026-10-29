# Component Patterns: Polaris `s-*` Web Components

This file is about **design-level component selection** inside Register A (the embedded admin surface): which primitive fits which content shape, how status color/tone semantics work, and a short list of real gotchas that trip people up in practice.

It is deliberately **not** an exhaustive prop/enum reference. For full, current component schemas and generated code, use the search-then-validate loop in the `shopify-polaris-app-home` and `shopify-polaris-admin-extensions` marketplace skills (if installed). If neither is installed, fall back to the `mcp__shopify-dev-mcp` tools directly: `search_docs_chunks` and `learn_shopify_api` to look up current props, and `mcp__shopify-dev-mcp__validate_component_codeblocks` to check any `s-*` markup before it ships. Never hand-write component props from memory when a validator is available.

One API-version fork to check before anything else, and it's scoped specifically to Admin UI extensions: on Admin UI extensions targeting API version `2025-07`, you write React components from `@shopify/ui-extensions-react/admin`, not `s-*` tags. On every other Admin UI extension API version (2025-10, 2026-01, 2026-04, unstable), you use `s-*` Polaris web components and nothing from `@shopify/ui-extensions-react/admin`. This is a hard either/or split, not a gradual migration. App Home (the iframe-based surface) and App Home UI extensions are a different surface entirely: they always use `s-*` Polaris web components, regardless of whatever `api_version` value shows up in their config, there is no React-vs-`s-*` fork on that surface at all. Everything below assumes the `s-*` path (shopify.dev/docs/api/admin-extensions).

## Ground rule: no import, globally registered

`s-*` elements (`s-page`, `s-section`, `s-badge`, `s-box`, and the rest) are custom elements registered globally by the platform. Never import from `@shopify/polaris`, `@shopify/polaris-react`, or `@shopify/polaris-web-components`, they don't exist for this surface. Use them directly as tags. Type support comes from `@shopify/polaris-types` and `@shopify/app-bridge-types` (shopify.dev/docs/api/polaris).

## Validator-confirmed gotchas

These are real mistakes a production app's lint/validator suite catches repeatedly. Confidence is marked per item against the research corpus, since some are well-documented and some are practical folklore that the docs don't spell out. When in doubt, validate with `mcp__shopify-dev-mcp__validate_component_codeblocks` rather than trusting any static list, including this one.

| Gotcha | Confidence | Why it happens |
|---|---|---|
| Prop names are camelCase, never kebab-case (`gridTemplateColumns`, `alignItems`, `borderRadius`, `paddingBlock`) | Confirmed repeatedly across all Polaris API docs | Muscle memory from CSS/HTML attributes pulls people toward hyphenated names |
| Boolean attributes (`disabled`, `loading`, `dismissible`, `checked`, `required`) accept shorthand or `{expression}`; string-keyword attributes (`padding`, `tone`, `variant`, `size`, `direction`, `background`) must always be actual quoted string values | Confirmed repeatedly, with explicit failing examples (`<s-box padding>` and `<s-stack gap={true}>` both fail TypeScript) | The two attribute families look identical at a glance, so people reflexively shorthand a keyword prop the same way they shorthand `disabled` |
| Inside an `@container` conditional value (e.g. `s-grid`'s `gridTemplateColumns`), spell columns out literally (`"1fr auto 1fr auto 1fr"`) rather than using `repeat()` | Confirmed by example: the Metrics card composition's real container-query value is `gridTemplateColumns="@container (inline-size <= 400px) 1fr, 1fr auto 1fr auto 1fr"`, never `repeat(...)`, while `repeat(2, 1fr)` does appear as a plain (non-container) value elsewhere | `repeat()` is valid CSS grid syntax and works in the plain form, so it looks safe to reuse inside the container-conditional value, it doesn't resolve the same way there |
| `s-text` emphasis is a `type="strong"` prop, not a `fontWeight` prop | Confirmed: every `s-text` example in the corpus uses `type` (e.g. `type="strong"`); no `fontWeight` prop ever appears on any component | Coming from CSS-in-JS or Tailwind backgrounds where `fontWeight` is the natural first guess |
| `s-spinner` needs an `accessibilityLabel`, and its `size` values follow a named token scale, not free-form t-shirt sizes | Partially confirmed: `accessibilityLabel` is directly confirmed; only `size="base"` is confirmed in the corpus, but the same token pattern (`small-100`, `small-200`, `small-300`, `small-400`, etc.) shows up on other components' spacing/size props, so treat `"small"` as unconfirmed and verify the exact enum before shipping | People reach for CSS-style `small`/`medium`/`large` instead of the platform's named token scale |
| `s-badge` `tone` is a closed enum built from the same status vocabulary as banners (something like `neutral`/`info`/`success`/`caution`/`warning`/`critical`) | Plausible but not exhaustively confirmed: `success`, `warning`, `critical`, `info`, and `neutral` all appear as real tone/status values elsewhere in the corpus; `"attention"` never appears anywhere as a valid value for any component | `"attention"` sounds like a natural tone name for "needs attention" copy, but it isn't part of the confirmed vocabulary, verify against current docs before using it |
| `key=` should not go directly on an intrinsic `s-*` element inside a rendered list; wrap it in a small keyed component instead | Not corroborated anywhere in the research corpus, this is carried over from a production app's lint suite only | Standard React list-key habits don't map cleanly onto remote-dom custom elements; treat this as a "verify in practice" item and validate the generated markup rather than assuming it |

## Choosing the right layout primitive for the content's actual shape

Three primitives get reached for constantly, and the most common slop-tell is picking the wrong one because it's the one you remember, not the one the content shape calls for.

**`s-stack`** is for one-dimensional flow: a row or column of things with consistent spacing between them. Reach for it when the content is fundamentally linear: a button group, a label next to a control, a short run of chips, the vertical rhythm inside a section. The research is explicit that for simpler linear layouts you should use stack instead of grid, don't reach for a two-dimensional primitive to solve a one-dimensional problem.

**`s-grid`** is for genuinely two-dimensional arrangements: a row of metric tiles, a gallery of template or app cards, a responsive layout that needs to collapse from multiple columns to one at a given container width. Real examples from the App Home patterns: a metrics card row using `gridTemplateColumns="@container (inline-size <= 400px) 1fr, 1fr auto 1fr auto 1fr"`, a template gallery using `gridTemplateColumns="repeat(auto-fit, minmax(155px, 1fr))"`, and an app-recommendation card internally using `gridTemplateColumns="auto 1fr auto"` to align a thumbnail, text block, and action button on one row. Grid doesn't support subgrid, nested grids are independent and don't inherit track sizing from their parent, so don't design a layout that depends on that.

**`s-table`** is for genuine tabular data: multiple records that share the same set of fields, where merchants need to scan, sort, or compare across rows and columns. Use `s-table-header-row` / `s-table-header` (with `listSlot="primary"` for the lead column, `listSlot="labeled"` plus `format="currency"` for typed value columns) / `s-table-body` / `s-table-row` / `s-table-cell`. Once the dataset is large or needs search, filtering, sort, bulk selection, or pagination, don't hand-roll that on top of a bare table, reach for the Index table composition instead, which layers those behaviors on in a Built-for-Shopify-compliant way. Full mechanics of that composition and index-page layout choices belong in this skill's layout reference, not repeated here.

Decision table for the content in front of you:

| Content shape | Reach for | Don't |
|---|---|---|
| A single number plus an up/down trend | Metrics card composition (an `s-badge` trend indicator alongside the number) | A full line chart to show one data point |
| Multiple records with the same fields, need to scan/sort/bulk-act | `s-table`, or the Index table composition for large sets | `s-grid` of cards (loses scannability and kills native sort/bulk actions) |
| A handful of dissimilar tiles (apps, templates, recommendations) | `s-grid` | `s-table` (forces heterogeneous content into artificial rows) |
| A short run of controls, labels, or buttons in one direction | `s-stack` | `s-grid` (two-dimensional overkill for one-dimensional content) |
| A page, list, or table with nothing to show | `s-empty-state` | A hand-built heading, graphic, and button. Filtered-to-zero uses the same component with "clear filters" copy, not onboarding copy |
| Trend over time, distribution, or proportion | A bespoke SVG or canvas chart (see below) | A table of raw numbers, or a row of stat tiles standing in for an actual trend |

## Badge and Banner tone semantics

Status color roles in the Shopify admin are semantically fixed, not decorative, and they map directly onto the `tone` values available on status components:

- **Success (green)** means a positive status or a successfully completed action. Nothing else. Don't use it to entice merchants toward an action.
- **Caution (yellow)** means paused, incomplete, or non-urgent information that needs attention eventually. Don't use it for announcements.
- **Warning (orange)** means in-progress, pending, or something that needs the merchant's attention now, it is the strongest non-blocking color role in the admin. Don't use it for "coming soon" or "under construction" messaging.
- **Critical (red)** means an action is impossible, blocked, or has resulted in an error. Reserve it for that. Don't use it to entice merchants either.

(shopify.dev/docs/apps/design/visual-design)

Practical split between the two components that carry these tones:

- **`s-badge`** is for compact, per-item status: a word or two that lives inline in a table row, next to a heading, or in a page accessory slot. It answers "what state is this one thing in" at a glance.
- **`s-banner`** is for page- or section-level state that needs an explanation and, often, an action. It persists until the merchant dismisses it (or resolves the underlying condition), and once dismissed in a session it should not reappear. The full taxonomy of banner vs. inline vs. toast, and when each is the right duration/interaction model for a given alert, is covered in this skill's alerts/feedback reference, don't rebuild that decision tree here, just note that badges and banners share the same tone vocabulary above.

Non-negotiable regardless of which component you pick: never rely on tone/color alone to carry the message. Pair it with text, and for banners, an icon too. A colorblind merchant should be able to tell a caution badge from a critical one from the words alone.

## When a bespoke SVG/canvas chart beats forcing data into a Polaris primitive

Polaris has no native chart component. That's not a gap to route around with a table of numbers, it's the one legitimate frontier for custom visual work inside Register A, and it's worth treating as a real creative decision rather than an afterthought.

Decision rule, based on what question the merchant is actually asking:

- "What are the individual records, and can I act on one of them?" → `s-table` or the Index table composition. Never a chart.
- "What's the current single number, and is it good or bad?" → the Metrics card composition (`s-badge` with a trend arrow icon and a success/warning/critical tone). Never a full chart for one number.
- "What changed, what's the shape of this over time, or how do these categories compare?" → this is the case a badge's trend arrow can't carry. Build a bespoke SVG or canvas chart.

When you do build one:

- Keep it inside a plain `s-section`/`s-box` container using the normal padding/border/radius tokens, so the chart's frame reads as native admin chrome even though its interior is fully custom.
- Give it complete interaction-state coverage like any other custom surface: a loading skeleton shaped like the eventual chart (not a generic spinner), an `s-empty-state` when nothing has been created (not a zero series), and an error state with an inline retry, not a silent blank.
- Treat it as your one signature moment for that page. If you're also inventing a custom empty-state illustration and a bespoke onboarding sequence on the same screen, you've spread the boldness too thin, pick the single place it earns its keep and keep everything else disciplined.

## Before you ship any of this

Run generated `s-*` markup through `mcp__shopify-dev-mcp__validate_component_codeblocks` (or the equivalent step inside the `shopify-polaris-app-home` / `shopify-polaris-admin-extensions` skills' search-then-validate loop) before treating it as done. Prop names, enum values, and slot names drift across API versions, and this file's job is to help you pick the right component for the content, not to be the source of truth for its exact schema.
