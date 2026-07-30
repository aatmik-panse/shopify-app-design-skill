# Layout and visual design

This file covers Register A only: the app's own pages inside Shopify Admin (App Home, settings, Admin UI Extension surfaces). Everything here assumes the Polaris `s-*` component system and its design tokens are the visual truth, not a canvas for custom brand styling. For the one place that changes (the App Store listing and any external marketing site), see this skill's marketing/App Store listing reference. For exact component prop names, enum values, and the validator-confirmed gotchas, see `reference/component-patterns.md`, and always run `validate_component_codeblocks` before shipping any markup shown here.

## Page structure and where actions live

A Shopify admin page has a fixed anatomy: app icon and name, page title, actions, overflow menu. Work within it rather than inventing a parallel header.

- The page title is set once, describes a single purpose, and should be the largest heading on the page (shopify.dev/docs/apps/design/visual-design). Don't repeat it as a redundant heading in the body.
- Primary and secondary actions live in the title bar's `primary-action` and `secondary-actions` slots, not floated inside the body. Exactly one primary-styled action per page, derived from the merchant's current state.
- Related secondary actions that don't fit the header get grouped into a menu (button with `commandFor` pointing at an `s-menu`), not spread across the page as a row of equal-weight buttons.
- Breadcrumbs or a back button go in `breadcrumb-actions` (or the page's built-in back affordance) so merchants can retreat without the browser back button.
- Status for the page as a whole (draft, syncing, needs attention) goes in the header's accessory slot as a badge, not as a banner competing with the primary action for the top fold.

```
<s-page heading="Campaigns">
  <s-badge slot="accessory" tone="warning">Draft</s-badge>
  <s-button slot="primary-action" variant="primary">Publish campaign</s-button>
  <s-button slot="secondary-actions" commandFor="more-actions">More actions</s-button>
  <s-menu id="more-actions">
    <s-button icon="duplicate">Duplicate</s-button>
    <s-button icon="export">Export</s-button>
  </s-menu>
  <s-link slot="breadcrumb-actions" href="/campaigns">Campaigns</s-link>
</s-page>
```

Inside the body, the same one-primary-action rule holds at the container level: a card or section that offers interactivity should have at most one primary-styled button in it (shopify.dev/docs/apps/design/layout). If a card has three buttons, one should read as the obvious next step and the rest should be secondary or tertiary in weight.

Table and index-row actions are a special case: never use primary-styled buttons inside a table row. Use a text button, a minor icon, or a dropdown menu instead (shopify.dev/docs/apps/design/layout). A table full of primary-colored row buttons reads as noise, not hierarchy.

Do / don't for action placement:

| Do | Don't |
|---|---|
| One primary action per page, in the title bar's `primary-action` slot | Two buttons on one screen both styled as primary, or both claiming the same intent |
| Group secondary actions into a menu once you have more than two or three | Row of five equal-weight buttons across the top of a card |
| Secondary/text-style actions inside table rows | Primary-colored buttons inside table rows (shopify.dev/docs/apps/design/layout) |
| Status badge in the header accessory slot | A banner announcing status sitting above the primary action on every visit |

## Section grouping and one/two-column layout

The `s-page` component ships three real layout modes for arranging content within the page: single-column, two-column, and settings (shopify.dev/docs/apps/design/layout). Pick based on what the page actually is, not on habit. Separately, `s-page` also takes an `inlineSize` setting, default-width or full-width, that controls how much horizontal space the page claims. That setting is independent of which of the three layout modes you're using; it's what the "Full-width page" pattern below is actually about, not a fourth peer layout mode alongside single-column, two-column, and settings.

**Single-column** is the default. It lets merchants scan top to bottom and focus on one obvious task, and it's the right choice for most homepages in a default-width page (shopify.dev/docs/apps/design/layout).

```
+-------------------------------------------+
| Page title            [Primary action]    |
+-------------------------------------------+
| Setup guide / banner (if relevant)         |
+-------------------------------------------+
| Section: metrics                           |
+-------------------------------------------+
| Section: recent activity                   |
+-------------------------------------------+
```

**Two-column** is for visual editors and content-dense pages where the merchant needs to see cause and effect at once, such as an editor with a live preview (shopify.dev/docs/apps/design/layout). It's also the right call when the page genuinely behaves like a dashboard, in which case use two-column equal-width (or three-column for even higher density) rather than forcing a dashboard into a single narrow column.

```
+---------------------+---------------------+
| Editor fields        | Live preview        |
| (primary/secondary)  |                      |
+---------------------+---------------------+
```

**Settings layout** is a thinner left column (title and description) paired with a wider right column (the actual form controls), used so merchants can quickly scan and find the setting they came for (shopify.dev/docs/apps/design/layout).

```
+-----------------+-----------------------------+
| Section title     | Form field                  |
| One-line context  | Form field                  |
+-----------------+-----------------------------+
```

**Full-width page** is the `inlineSize="full-width"` setting on `s-page`, used for resource index pages where merchants deal with lists of data that have many columns (shopify.dev/docs/apps/design/layout). It's typically paired with a single-column arrangement of content, most often a table or the Index table pattern, rather than with the two-column or settings arrangement.

The rules that keep these from being interchangeable:

| Situation | Use | Don't |
|---|---|---|
| App homepage, no parent/child relationship between sections | Single-column, default width | Two-column primary/secondary layout on the homepage (Built for Shopify design requirements) |
| Visual editor with real-time preview | Two-column primary/secondary, default width | A single column that hides the preview behind a tab or a scroll |
| Page behaves as a dashboard | Two-column equal-width, or three-column for higher density | Forcing dashboard content into single-column full-width |
| Resource index (products, orders, a custom list) | Single-column, full-width | Single-column full-width page for anything that isn't list data, it feels out of place on large displays (shopify.dev/docs/apps/design/layout) |
| Settings and configuration | Settings layout (thin left, wide right) | A flat single column of unlabeled fields with no grouping |
| Content that isn't actually editable | Single-column | An "annotated" two-column layout implying editability that isn't there (Built for Shopify changelog) |

Also don't put a two-column layout inside a full-width page. It makes content harder to parse on larger displays (Built for Shopify design requirements changelog), because the columns end up too wide to read comfortably.

Within any of these, the majority of content should live in a container such as a card, since containers create the visual structure and rhythm merchants rely on to parse a page (shopify.dev/docs/apps/design/layout). Headings, simple text, and images don't strictly need to be wrapped, but don't leave paragraphs of body text floating directly on the page background, it hurts legibility and scanning (shopify.dev/docs/apps/design/layout). The inverse failure mode is just as real: nesting a card inside a card inside a section because "container" became the reflexive default rather than a deliberate grouping decision. If two adjacent cards share no real elevation or boundary purpose, they should be one section, not two containers.

## Spacing and density

The Shopify admin is built on a 4px spacing grid (shopify.dev/docs/apps/design/layout). Use `s-stack` to get consistent spacing without hand-rolling margins, and `s-section` to segment content into logically distinct groups.

- Use looser spacing for low-density layouts (a homepage, a settings page) and tighter spacing for high-density layouts (a data table, a bento grid of small tiles).
- Don't mix densities on a single page. A page that's airy at the top and cramped at the bottom feels disjointed (shopify.dev/docs/apps/design/layout).
- This skill can't hand you exact pixel or token values for "tight" vs "loose" spacing scales because the research corpus only surfaced named tokens (`base`, `loose`, `tight`, `small-200`, and similar), not a numeric scale. Treat those as directional guidance and confirm the actual enum a given prop accepts with `reference/component-patterns.md` or `validate_component_codeblocks` before writing the value into code.

## Creative layout composition within the token system

This is where real design judgment lives in Register A: not new colors or fonts, but how you arrange the grid you're given.

**Asymmetric grid and stack composition.** `s-grid` follows CSS grid conventions: you can size columns unevenly (`"2fr 1fr"`), and give individual items explicit spans so one tile reads as the anchor and the rest as supporting detail, rather than defaulting every tile to identical width. A 12-column system works the same way: pairing two items each spanning 6 columns produces an even two-up layout, while pairing a 4-span item with an 8-span item produces a deliberate primary/secondary asymmetry.

```
<s-grid gridTemplateColumns="2fr 1fr" gap="base">
  <s-box padding="base" border="base" borderRadius="base">Primary content</s-box>
  <s-box padding="base" border="base" borderRadius="base">Supporting detail</s-box>
</s-grid>
```

**Bento-style tile composition.** Building a small dashboard as a set of unevenly-sized tiles inside one `s-grid` (one large tile for the thing that matters most, several small tiles around it) is a legitimate way to create hierarchy without leaving the token system. Use `gridColumn="span 2"` (or higher) on an item to make it visually dominant relative to its siblings:

```
<s-grid gridTemplateColumns="1fr 1fr 1fr" gap="base">
  <s-grid-item gridColumn="span 2">
    <s-box padding="base" border="base" borderRadius="base">Featured metric</s-box>
  </s-grid-item>
  <s-box padding="base" border="base" borderRadius="base">Secondary metric</s-box>
  <s-box padding="base" border="base" borderRadius="base">Secondary metric</s-box>
</s-grid>
```

Two constraints worth knowing before you lean on this pattern: grid doesn't support subgrid, so a nested grid is independent and won't inherit track sizing from its parent, and inside an `@container` query value, docs examples spell columns out literally (`"1fr 1fr"`) rather than using `repeat()`, so treat `repeat()` inside a container-query value as unverified until you've checked it with `validate_component_codeblocks`. Exact prop names and enum values for `s-grid` / `s-grid-item` belong in `reference/component-patterns.md`, not here.

**Responsive stacking via container queries.** Wrap a grid in `s-query-container` and key the column count to the container's own width, not the viewport, so the same component behaves correctly whether it's rendered full-width or inside a narrower nested layout:

```
<s-query-container>
  <s-grid gridTemplateColumns="@container (inline-size > 500px) 1fr 1fr, 1fr" gap="base">
    <s-box padding="base">Tile A</s-box>
    <s-box padding="base">Tile B</s-box>
  </s-grid>
</s-query-container>
```

**When a bespoke chart is the right creative move.** Polaris's own vocabulary for quantitative content stops at Metrics cards (a number, a trend arrow, an optional comparison) and Index tables. Nothing in Polaris's composition catalog is a chart or graph primitive. That gap is real, and it's the one place a hand-built SVG or canvas visualization is not just permitted but often the correct answer, because a merchant trying to understand a trend over time or a distribution across categories genuinely needs more than a single number with an arrow next to it.

Ground rules for building one:

- Build it because a decision depends on the shape of the data (is this trending up or down, where's the outlier), not because a chart looks more finished than a number. A chart that doesn't change what the merchant does next hasn't earned its pixels.
- Use Polaris's semantic color roles for any status encoding inside the chart (a line going into "needs attention" territory should use the same orange the rest of the app uses for that meaning), not an arbitrary chart-library default palette.
- Give it full interaction-state coverage: loading skeleton shaped like the eventual chart, empty state, error state with retry, and a populated state. A chart that only has a happy path is not done.
- Respect `prefers-reduced-motion` for any animated transitions between data states.
- Treat it as the one signature moment for that surface. If you build a bespoke chart, keep everything else on the page disciplined and token-standard around it. Don't also add a custom illustration and a custom motion sequence on the same screen, that's spreading boldness thin instead of spending it in one place.

## Illustration and empty-state art

Illustrations are one of the few places Polaris explicitly invites visual personality: they give the app character and should follow a consistent style to strengthen recognition, as long as that style stays consistent across the app rather than looking like stock art bolted on per screen (shopify.dev/docs/apps/design/visual-design). Low-resolution photos or images read as a lack of care and should be avoided outright (shopify.dev/docs/apps/design/visual-design).

The Empty state composition is the primary place illustration earns its keep: for moments where there's genuinely nothing to show yet, pair a clear explanation of what will eventually appear with a prominent call-to-action, rather than leaving merchants looking at a blank card. Use cases include first-item onboarding, no search or filter results, and feature-activation prompts. An empty state is not the same thing as a filtered-to-zero state, the latter needs a "clear filters" affordance, not onboarding illustration and copy.

Treat one well-made empty-state illustration (or one well-orchestrated onboarding sequence) as your signature moment for a given surface, the same way a bespoke chart can be. Don't illustrate every empty state, every onboarding step, and every success toast on the same page. Pick the one that matters most to first impressions and invest there.

## Color: semantic status only, and where that limit genuinely loosens

Inside Register A, color is not a branding tool. It's a fixed status vocabulary, and the roles don't bend for taste:

| Color | Meaning | Never use it for |
|---|---|---|
| Green (`--p-color-text-success`) | Positive status or a successfully completed action | Enticing merchants or drawing attention for its own sake |
| Yellow (`--p-color-bg-fill-caution`) | Paused/incomplete status, or non-urgent information needing attention | Announcements |
| Orange (`--p-color-bg-fill-warning`) | In-progress/pending status, or something that needs the merchant's attention. The strongest non-blocking color role in the admin | "Under construction" or "coming soon" messaging |
| Red (`--p-color-text-critical`) | An action that's impossible, blocked, or has resulted in an error | Enticing merchants, or anything short of a genuine blocker or error |

(shopify.dev/docs/apps/design/visual-design; Built for Shopify design requirements restate the same four roles with the same hex/rgba values.)

Rules that apply everywhere color appears:

- Never rely on color alone to carry meaning. Every alert, badge, or status indicator pairs color with text or an icon (shopify.dev/docs/apps/design/visual-design).
- Background-to-text contrast must hit at least 4.5:1 (WCAG AA); check it with a real contrast tool rather than eyeballing it (shopify.dev/docs/apps/design/visual-design).
- Don't use button colors, background colors, or fonts that diverge from the admin's own palette (a green or purple primary button, a black page background) purely to "look like the brand." Built for Shopify calls this out directly as a rejection reason.
- Don't use the Shopify Sidekick icon or Shopify's own "magic purple" to denote an AI feature in your app, that's brand confusion with Shopify itself, not brand expression for your app (Built for Shopify requirements).

Where the limit genuinely loosens, inside Register A:

- **Imagery and illustration style**, not chrome or interactive-element color. Shopify's own guidance is explicit that branding should be expressed with a light touch, primarily through imagery and illustration, while layout and common components stay consistent with the rest of the admin (shopify.dev/docs/apps/design/user-experience/marketing). A distinctly-styled illustration set is legitimate brand expression. A distinctly-colored button is not.
- **The app icon**, which is a bounded, spec'd surface (fixed dimensions, fill ratio, margin) rather than a page you're theming, and is the one place bold, recognizable brand color is expected (shopify.dev/docs/apps/design/visual-design).

Full brand color freedom, distinctive typography, saturated palettes, marketing hero treatments, only exists in Register B: the App Store listing (icon, screenshots, feature graphic, listing copy) and any external marketing site for the app. See this skill's marketing/App Store listing reference for that register's rules in full. Bringing that freedom into Register A (a custom display font on the homepage, a saturated brand-color wash behind the header, a welcome banner styled like a marketing hero) is the single most common way an otherwise well-built app stops reading as a real Shopify app.

## Quick design-read before you touch layout

Before composing a new page or section, state in one line: what surface is this (App Home vs Admin Extension vs Store listing), what's the merchant's real state, and what's the one primary next action. Then sketch the section plan as prose plus a small ASCII wireframe, check it against the column/density/color rules above, and only then write the `s-*` markup. Validate the result with `validate_component_codeblocks` and look at it in the actual Shopify Admin at both a normal and a narrow viewport before calling it done.
