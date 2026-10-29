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

Inside the body, the same one-primary-action rule holds at the container level: a section that offers interactivity should have at most one primary-styled button in it (shopify.dev/docs/apps/design/layout). If it has three buttons, one should read as the obvious next step and the rest should be secondary or tertiary in weight.

Table and index-row actions are a special case: never use primary-styled buttons inside a table row. Use a text button, a minor icon, or a dropdown menu instead (shopify.dev/docs/apps/design/layout). A table full of primary-colored row buttons reads as noise, not hierarchy.

Do / don't for action placement:

| Do | Don't |
|---|---|
| One primary action per page, in the title bar's `primary-action` slot | Two buttons on one screen both styled as primary, or both claiming the same intent |
| Group secondary actions into a menu once you have more than two or three | Row of five equal-weight buttons across the top of a section |
| Secondary/text-style actions inside table rows | Primary-colored buttons inside table rows (shopify.dev/docs/apps/design/layout) |
| Status badge in the header accessory slot | A banner announcing status sitting above the primary action on every visit |

## Page width and layout patterns

`s-page` `inlineSize` is `small`, `base` (the default), or `large`. `large` is the wide page, with whitespace. These are widths, not layout modes (shopify.dev/docs/api/app-home/latest/web-components/layout-and-structure/page).

How the body is arranged is a pattern on top of that width (shopify.dev/docs/apps/design/layout, shopify.dev/docs/apps/build/app-home/migrate-from-polaris-react/layout):

| Situation | Use |
|---|---|
| Most homepages | A single column of sections at `inlineSize="base"` |
| Dashboard | `inlineSize="large"` and an `s-grid` of two or three equal columns |
| Resource index | `inlineSize="large"`, so a many-column table has room |
| Supporting sidebar | `slot="aside"` on a direct child of `s-page`. The aside renders only when `inlineSize` is `base`. Keep the primary task in the default slot |
| Settings | One `s-section` per group, often on `inlineSize="small"`. When a title and description need to sit beside the controls, use an annotation `s-grid` (narrow text column, wider fields) that stacks to one column when narrow |
| Visual editor with a live preview | Two columns, so the preview stays on screen |

`small` is for a focused form. Don't put primary controls in the aside.

Most content sits in an `s-section`. That is the card-like container; Polaris web components have no card element (shopify.dev/docs/apps/build/app-home/migrate-from-polaris-react, shopify.dev/docs/api/app-home/latest/web-components/layout-and-structure/section). Use a bordered `s-box` for a tile that stands alone in a grid (shopify.dev/docs/api/app-home/latest/patterns/templates/homepage). Don't nest sections for decoration: nesting sets the heading outline, and a nested section does not draw its own card. A heading or a line of plain text needs no container of its own (shopify.dev/docs/apps/design/layout).

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

**Responsive stacking via container queries.** Wrap a grid in `s-query-container` and key the column count to the container's own width, not the viewport, so the same component behaves correctly whether its container is wide or narrow:

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

Illustration earns its keep on `s-empty-state`: a clear explanation of what will appear, plus one prominent action, rather than a blank page. A filtered-to-zero result needs "clear filters" copy, not onboarding illustration.

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
