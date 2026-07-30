# Creative Playbook: Where Shopify Apps Actually Have Room to Be Great

This file assumes you already know the two registers (see SKILL.md), the token system, the `s-*` component gotchas (see `reference/component-patterns.md`), layout composition rules (see `reference/layout-and-visual-design.md`), and the non-negotiable accessibility bar (see `reference/accessibility.md` or equivalent). None of that is repeated here. This file answers a narrower question: given all of that is fixed, where do you actually spend creative effort, and how much?

Register A (the embedded admin surface) is not a place where you get to invent a visual language. It is a place where you get to make better decisions than the last person who built a Shopify app. The creative work is real, it just lives at a different altitude: information architecture, sequencing, state completeness, data visualization, motion, illustration, and placement. Master those and an app feels distinctive and cared-for without ever looking like it's wearing a costume. Skip them and you get exactly what every reviewer, merchant, and Built for Shopify auditor has seen a thousand times: a competent, forgettable grid of cards.

## The six places creative judgment actually lives

### 1. Bespoke data visualization

Polaris has no native chart components. This is not a gap to route around with a third-party chart library styled to look Polaris-ish — it is the single most legitimate creative frontier inside Register A. A merchant looking at a well-drawn, purpose-built chart inside an `s-section` reads it as competence, not as branding.

**Surface scope: this is an iframe-based App Home pattern, full stop.** Everything below — the raw SVG/canvas element, the plain `<div>` in the example markup — depends on rendering inside your app's own iframe, where arbitrary HTML/CSS/JS is fair game. Admin UI Extensions and the App Home UI Extension surface render through remote-dom instead: only registered `s-*` custom elements work there, SVG is explicitly unsupported as a bundled asset, and the whole compiled bundle is capped at 64KB. None of that leaves room for a hand-drawn chart. If a feature calls for a data visualization inside a Block, Action, or the App Home UI extension (see #4), reach for a Metrics-card-style composition instead — a number plus an `s-badge` trend indicator (`tone="success"`/`"warning"`/`"critical"`, `icon="arrow-up"`/`"arrow-down"`) — and save the bespoke chart for the iframe surface.

Rules for doing this well, inside that iframe:

- Build the chart as plain SVG or canvas, sized to its container, not as a component trying to imitate `s-*` styling. It should look like a chart a Shopify admin page would draw, not like a marketing dashboard screenshot.
- Use Polaris's own semantic color roles for chart state, not an arbitrary palette: green only for positive/successful trends, orange for in-progress/needs-attention, red only for problems, yellow for paused/non-urgent. Inventing a five-color categorical palette for a chart inside admin is a Register B habit leaking into Register A — pick one accent hue that reads as "this app's data," and keep everything else neutral grays.
- Every chart needs a text-equivalent. A sparkline or trend line is decoration on top of a number that must also exist as legible `s-text`/`s-heading` content — screen readers get nothing from an SVG path.
- Wrap the chart in an `s-query-container` and drive its layout off container width, not viewport width, so it behaves correctly inside a narrow app-embed column or a wide two-column dashboard alike — both still inside the iframe surface this pattern is scoped to:

```html
<s-query-container containerName="chart-frame">
  <s-box padding="base" border="base" borderRadius="base">
    <s-stack direction="block" gap="small-200">
      <s-heading>Visibility over time</s-heading>
      <!-- your custom SVG/canvas element goes here, sized via container queries -->
      <div class="app-trend-chart" role="img" aria-label="Visibility score rose from 42 to 61 over the last 30 days"></div>
      <s-text tone="subdued">61 / 100, up 19 points in 30 days</s-text>
    </s-stack>
  </s-box>
</s-query-container>
```

- A chart that doesn't change a decision hasn't earned its space (see the anti-slop list below on vanity metrics). Before building a bespoke visualization, state in one sentence what action a merchant takes differently because they saw it. If you can't, it's decoration, and decoration competes with your one signature moment (see #6).
- Loading and empty states for a custom chart are not optional. A chart with zero data points needs its own designed empty state (see #3), not an empty SVG canvas. A chart that's loading needs a skeleton shaped like the chart (see `reference/alerts.md` for the general feedback-state vocabulary this borrows from), not a generic spinner floating in a box.

This is the skill's canonical, detailed treatment of the bespoke-chart decision; `reference/component-patterns.md` cross-references this section rather than repeating it.

### 2. Tasteful authored motion, always behind one gate

Motion is legitimate creative territory in Register A specifically because Polaris ships almost none of it. A number that counts up, a checklist item that settles into place when checked, an empty-state illustration that breathes gently while idle — these make an app feel considered rather than static, without touching a single Polaris visual token.

The rule that makes this safe, stated once so you never have to re-derive it:

```css
@media (prefers-reduced-motion: no-preference) {
  .app-signature-moment {
    animation: settle 320ms ease-out;
  }
}
```

Every custom animation you author lives inside a `prefers-reduced-motion: no-preference` block, full stop. No exceptions for "it's subtle" or "it's just an opacity fade." This is both a WCAG expectation and an explicit Shopify accessibility requirement for media controls, and it costs you nothing to honor.

Where authored motion is appropriate:
- Around and inside component **slots** you own (a custom empty-state illustration, a custom chart drawing itself in, a checklist item transitioning to "done") — never on the internal DOM of an `s-*` component itself, which you don't control and shouldn't try to reach into.
- State transitions that reinforce progress (a Setup guide task checking off, a metric updating after a scan completes) — motion as feedback, not motion as decoration.
- Never on page load for a returning merchant's primary content. If a merchant opens the app and its dashboard fades/slides everything into view every single time, that's not delight, that's friction with an animation-shaped costume — this is one of the concrete "banner/hero dominates the top fold" anti-slop failure modes described below, just executed with motion instead of a hero block.

Where authored motion is not appropriate: simulating Polaris component transitions, animating anything that will be perceived as loading-state deception (don't animate a fake progress bar that isn't tied to real progress), or motion competing with the one signature moment on the page (see #6 — one thing moves with intention, not five things moving a little).

### 3. Empty-state illustration

The Empty state composition (see corpus: shopify.dev's App Home patterns) already gives you the mechanics — clear explanation plus a prominent call-to-action. What it doesn't give you is a visual, and that gap is intentional: illustration for empty/onboarding moments is named explicitly as legitimate creative territory.

Do:
- Keep one consistent illustration style across every empty state in the app. A merchant who sees three different illustration styles in one app reads it as unfinished, not as variety.
- Illustrate the *before*, not an abstract mascot. Show the shape of what will appear once there's data — a faint outline of the chart that will populate, a ghosted row of the table that will fill in — rather than a generic "empty box" cartoon unrelated to the feature.
- Distinguish a true empty state (nothing has happened yet, needs onboarding-style guidance) from a filtered-to-zero state (something exists, current filters just hide it). The latter gets a "clear filters" affordance and neutral copy, never the illustration and onboarding tone of the former — conflating these is one of the most common Register A mistakes.
- Follow Polaris illustration guidelines for admin consistency (see `reference/layout-and-visual-design.md`) rather than importing an illustration style built for the marketing site. This is the one place illustration belongs in Register A; don't import Register B's hero-illustration scale or saturation here.

Don't: use a low-resolution stock photo or an AI-generated image that doesn't match a consistent style — this reads as a lack of care faster than almost anything else on the page.

### 4. Placement via Admin UI Extensions — creativity as information architecture, not visual styling

This is the creative lever most Shopify-app builders never touch, because it isn't about how something looks, it's about *where it lives*. The six admin extension target types (Action, Selection action, Block, Configuration, Print action, Runnable) are a placement palette. Choosing the right one for a workflow is a genuine design decision with real leverage — often more leverage than any amount of visual polish on the app's own homepage.

Concretely:
- A merchant fixing a product's SEO title shouldn't have to leave the product page, find your app in the nav, search for that product again, and edit it there. An inline `admin.product-details.block.render` block puts the fix exactly where the problem was noticed. This is the difference between a tool that respects a merchant's workflow and one that interrupts it.
- A bulk operation (batch re-optimize, bulk tag, multi-item export) belongs in a Selection action on the index page's "More actions" menu when merchants have selected several resources — not buried in your app's own bulk-tools page that requires re-selecting the same items a second time.
- A one-off, page-specific action (regenerate this one product's content, run a single audit) belongs in an Action target's "More actions" menu on that resource's detail or index page — again, in situ, not in your app's nav.
- Document generation (a spec sheet, a compliance label, a custom packing slip) belongs in a Print action target, which appears in the Print menu specifically — don't repurpose a Block or Action for something merchants expect to find under Print.
- A bundle/customizable-product configuration UI belongs in a Configuration target (`admin.product-details.configuration.render`), which Shopify only surfaces when the product is actually configured as a bundle — this is placement doing validation for you for free.
- Remember the constraint that comes with this power: a Block extension can't be fully hidden at runtime — returning null only collapses it, it doesn't remove it from the page — so design the collapsed state on purpose, and remember merchants must manually add and pin a block before it appears at all, so a first-run nudge from your app's homepage ("Add the [Feature] block to your product pages") is often necessary.

The creative judgment here is entirely about *reducing the distance between where a merchant notices a problem and where they can act on it*. That's an information-architecture insight, not a visual one, and it's available to you regardless of how constrained the token system is. Treat "should this live in the app's own page, or should it live in a Block/Action/Print-action right where the merchant already is" as a real design question for every feature, not an afterthought.

### 5. Layout composition within the token system

Grid arrangement, section grouping, and asymmetric spans inside `s-grid`/`s-stack` are real creative choices even though every individual component is fixed. This ground is covered in depth in the layout reference (see `reference/layout-and-visual-design.md`) — the point to hold onto here is that composition (what groups with what, what gets a two-column emphasis versus a single full-width row, what density a section runs at) is a legitimate design decision, not a constraint to route around. Don't reach for custom CSS grid overrides to get a layout the token system already supports through composition.

### 6. One signature moment — spend your boldness in one place

Every surface (a homepage, a settings page, an onboarding flow, a single Block extension) gets exactly one place where you spend your creative budget: one bespoke chart, one genuinely delightful empty-state illustration with authored motion, or one well-orchestrated onboarding sequence. Everything else on that surface stays disciplined — standard compositions, standard spacing, standard tone.

This is a real constraint, not a suggestion. A homepage that has a beautiful custom visibility-score chart *and* an animated illustration *and* a custom-styled setup guide *and* a bespoke onboarding sequence doesn't read as ambitious, it reads as unfocused — every one of those elements dilutes the others and the page stops feeling like a considered admin tool. Pick the one moment that actually matters for this surface's job (usually: the thing that answers the merchant's single biggest question the fastest) and let it carry the page. Leave the setup guide, the tables, the forms, and the alerts exactly as boring and correct as the compositions intend them to be.

Concretely, before building a page, name the signature moment in one sentence: "the visibility-trend chart is the one bespoke element on this homepage; the setup guide, metrics cards, and callout card around it use stock compositions unmodified." If you can't name it in one sentence, you haven't decided yet, and the page will end up either flat (no moment at all) or noisy (competing moments).

## Process: design read, sketch, critique, build, critique again

Before writing a single line of JSX for a new page or flow, work through this sequence in order. Skipping straight to implementation is how generic layouts happen.

1. **State the design read in one line.** Surface type (App Home vs. Admin Extension vs. Store listing), the merchant's real state (first-run vs. returning vs. mid-task, zero data vs. data), and the one primary next action this surface exists to drive. Example: "App Home, returning merchant with an active scan history, primary action is reviewing the 3 flagged issues from last night's run."
2. **Sketch the section/layout plan in prose plus a small ASCII wireframe.** This forces you to commit to hierarchy before component details distract you. Example:

```
[ status banner: only if something needs attention, else absent ]
[ signature moment: visibility trend chart, full width ]
[ 3 flagged issues — index-style list, each row = one action ]
[ setup guide — collapsed by default, only if setup incomplete ]
[ footer help link ]
```

3. **Check the plan against the anti-slop list below** before writing code. Is there a flat grid of equal-weight cards with no hierarchy? A vanity metric with no attached decision? A hero banner eating the top fold for a returning user? Fix the plan, not the pixels, if any of these show up.
4. **Only then implement**, using the stock Templates/Compositions as the default and deviating only where the signature moment calls for it. Validate any hand-written `s-*` markup with `validate_component_codeblocks` before treating it as done — prop names and enum values are easy to get subtly wrong, and this skill doesn't own the exhaustive component reference (that's `shopify-polaris-app-home` / `shopify-polaris-admin-extensions`, or the raw `mcp__shopify-dev-mcp` tools when those aren't installed).
5. **Screenshot and review the real result** in actual Shopify Admin, not just in isolation — at a merchant's typical desktop viewport and at a narrow one. A layout plan that looked right in the ASCII sketch can still collapse badly at a real breakpoint; this step catches what the sketch can't.

## Pre-ship self-critique checklist

Run this after the page works and before calling it done. Every "no" here is a reason to go back a step, not a reason to ship anyway.

- [ ] **One primary action.** Is there exactly one dominant, state-derived primary action on this screen — never two CTAs carrying the same intent?
- [ ] **Signature moment named.** Can you say, in one sentence, which single element is this surface's deliberate creative spend, and is everything else around it disciplined by comparison?
- [ ] **State completeness.** Does every custom surface (not just the happy path) have a designed loading state (shaped like the loaded layout, not a generic spinner), empty state (distinct from filtered-to-zero), error state (a banner with an inline Retry, never a silent fallback to empty), and success state?
- [ ] **Motion gated.** Does every authored animation live inside `prefers-reduced-motion: no-preference`, with nothing depending on motion to convey state?
- [ ] **Contrast checked.** Does body text hit at least 4.5:1 and large text at least 3:1, especially anywhere you used Polaris's "subdued" tone — it's the easiest trap on this neutral palette?
- [ ] **Keyboard and focus.** Is every interactive element, including anything inside a custom chart or illustration, reachable and visibly focused via keyboard alone?
- [ ] **z-index deliberate.** If you built a custom overlay/tooltip/popover alongside `s-*` components, does it sit on a deliberate z-index scale rather than an arbitrary number picked to "just work"?
- [ ] **Copy self-audit.** Reread every visible string out loud. Cut anything that is grammatically broken, has an unclear referent ("it," "this" with no clear antecedent), or is cute/AI-ish wordplay that doesn't survive a second read. Then check it against German-length expansion (roughly +30%) — does any string only fit at English length?
- [ ] **Placement re-checked.** For every action a merchant takes, could it live closer to where the problem was noticed via an Admin UI Extension target, instead of requiring a trip to the app's own page?
- [ ] **Dismissal persists.** Does every dismissible banner, setup guide, and callout card actually remember it was dismissed (not just visually hide until next reload)?
- [ ] **Real-environment review done.** Have you actually looked at this in Shopify Admin, at both a normal and a narrow viewport, rather than only in isolation?

## The Shopify-admin slop test (Register A specific)

A marketing-site slop test asks "does this look like every other AI-generated landing page." That's a different set of tells from what clichéd Register A output actually looks like. Run this test on any admin surface before shipping:

| Slop tell (Register A) | What it looks like | The fix |
|---|---|---|
| SaaS dashboard collage | A flat grid of equal-weight KPI cards with no hierarchy tied to what the merchant does next | Rank by decision-relevance; give the one number that changes an action visual priority over the rest |
| Vanity metrics | A number shown with no action attached to it | Cut it, or attach the action it should trigger |
| Hero eating the fold | A marketing-style welcome banner dominating the top of the screen for a *returning* user on a working surface | That space belongs to live state and the next action; move brand moments to onboarding or Register B |
| Cosmetic setup guide | A checklist that isn't actually gated to real install/data state | Gate every step to a real, checkable condition; mark complete only when it's true |
| Banner-blindness | Standing, non-urgent information living in a dismissible banner | Move it into a plain section; reserve banners for genuinely time-bound or actionable alerts |
| Reflexive nesting | Cards used as the default container even with no elevation/grouping reason, including cards nested inside cards | Use a section or plain grouping unless there's a real reason to elevate |
| Diluted boldness | Three "creative" elements on one page (a chart, an illustration, a custom onboarding flourish) all competing | Pick the one signature moment for this surface; discipline everything else |
| Same person as everyone else's help modal | A support/promo experience that auto-pops on load, on a timer, or after an unrelated action | Never auto-surface promotional content; keep it dismissible and merchant-triggered only |

This table synthesizes the anti-slop patterns named throughout this skill's other reference files — treat it as a working checklist distilled from that guidance, not a one-off opinion. If a page fails more than one row here, it hasn't actually used the creative freedom this file describes — it's just added surface area without judgment.

## The boundary that makes all of this safe

Everything in this file operates entirely within Register A's constraints: the token system, the `s-*` component set, and Built for Shopify's design requirements are never negotiable, no matter how good the creative idea is. If an idea requires a custom font, a saturated brand-color wash, or a hand-rolled override of a Polaris component's internals to work, it isn't a Register A idea — it belongs on the App Store listing or marketing site instead, where that freedom is not just allowed but expected. The skill of this file is finding real creative leverage *inside* the fence, not arguing for a taller fence.
