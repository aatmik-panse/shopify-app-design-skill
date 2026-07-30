# Navigation

Navigation is how a merchant knows where they are, what they can do next, and how to get back. In Register A this is almost entirely a solved-shape problem: Shopify defines three navigation surfaces (App nav, App header, Page title) and a small set of secondary patterns (tabs, filters, breadcrumbs). Your job is picking the right one for the job, not inventing a new one. See `reference/component-patterns.md` for exact `s-*` component props; this file covers the decisions, not the API surface.

## The three navigation surfaces

Shopify names three places navigation elements live (shopify.dev/docs/apps/design/navigation):

1. **App nav** — the merchant's way to move between pages of your app. Lives in the Shopify admin sidebar on desktop and collapses into the header/title-bar dropdown on mobile.
2. **App header** — app icon and name, page title, page-level actions, and the overflow menu.
3. **Page title** — the heading identifying the current page, plus its primary/secondary actions.

Rely on the relationship between the app nav and the app body to move merchants around. Don't invent a fourth navigational structure alongside these (shopify.dev/docs/apps/design/navigation).

## Information architecture: draw the map before the routes

- Use the fewest possible categories to define what your app does (shopify.dev/docs/apps/design/navigation). If you're reaching for a fourth or fifth top-level nav item, that's a signal to consolidate, not to add a "View more."
- Navigation item labels are nouns, not verbs: "Templates," "Reports," "Settings," never "Manage templates" or "View reports." This keeps the sidebar scannable and mirrors how the rest of the admin is labeled.
- More than 7 App nav items and item 7+ truncate into a "View more" button automatically. Don't design past this threshold and expect every item to stay visible; if you're near it, that's another signal to consolidate categories rather than lean on the overflow.
- Don't replicate app nav content inside the app body (a homepage card that just re-lists the same links as the sidebar is redundant, not helpful).
- Don't place your main navigation in the page header. The header is reserved for in-page actions; putting nav there misleads merchants about what it's for.
- Don't send merchants outside of the Shopify admin for key actions or during primary workflows. If a feature can only exist as an external page, ship a simplified version inside the admin and treat the external surface as supplementary (this is also a Built for Shopify design requirement, not just a courtesy).

### App naming: two different character limits, don't confuse them

- The **app name shown in the App nav / admin sidebar** (set via Dev Dashboard > Versions > Create a version) is capped at 20 characters before it truncates with an ellipsis — and Built for Shopify requirement 4.1.3 treats that truncation as a rejection reason, especially when the app is pinned. Keep it short and don't put a description in it.
- The **Shopify App Store listing name** (Register B, see `reference/marketing-surfaces.md` if present) is capped separately at 30 characters and can be longer/more descriptive than the nav name.
- These names can differ. Optimize the nav name for fitting the sidebar; optimize the listing name for search and positioning.

## Primary navigation: the App nav

Use the App Bridge app nav web component (`s-app-nav`) to integrate your primary navigation into the Shopify admin's own nav menu, rather than building a separate in-app navigation menu (Built for Shopify 4.1.4). This is not optional polish: a custom nav bar bolted into the app body reads as "not a real Shopify app" and is an explicit rejection reason.

```
<s-app-nav>
  <s-link href="/templates">Templates</s-link>
  <s-link href="/reports">Reports</s-link>
  <s-link href="/settings">Settings</s-link>
</s-app-nav>
```

- The app name in the sidebar already acts as the home link. Don't add a separate nav item that redirects to your homepage as well; if your homepage lives at a non-root path, add one `s-link` with `rel="home"` to point the app name there instead, which also hides that link from the rendered menu. Only one link should carry `rel="home"`.
- Navigating to a sub-page must highlight the relevant parent nav item. A merchant three levels deep in Settings should still see "Settings" marked current in the sidebar.
- Navigation icons render gray when inactive and green when active, and are auto-cropped to a 4px border radius, so don't pre-round icon corners yourself.
- Exact current props/behavior for `s-app-nav` (there are two documented variants: the App Bridge web component and the newer App Home UI extension navigation component, with slightly different `rel="home"` semantics) should be validated against the live component reference or with `validate_component_codeblocks` before shipping, since the two versions aren't guaranteed identical.

### Accessibility baseline for the App nav

These apply whether you're relying on `s-app-nav` or building any custom navigation landmark inside your own pages (e.g. a footer link list):

- Wrap navigation areas with the `nav` element.
- Use `aria-current` (typically `aria-current="page"`) to communicate the current location, not a bespoke "active" class alone.
- Do not use `role="menu"` or `role="menuitem"` for navigation — those roles are for application menus (dropdowns, command menus), and screen readers announce them with different, misleading expectations for what a straightforward link list should do.
- If any part of your nav is a drop-down (e.g. a "More" flyout), it needs `aria-expanded` for collapsed/expanded state and `aria-controls` referencing the controlled container; Enter/Space should open it while focus stays on the launcher, Tab should move focus into the first item, and Esc should collapse it and return focus to the launcher (shopify.dev/docs/apps/build/accessibility).

## The App header: title, actions, breadcrumbs

The app header contains the app icon/name, the page title, page-level actions, and the overflow menu. The overflow menu is reserved for support information only ("About this app," "Support") and is not customizable — don't try to smuggle extra actions into it. On mobile, the app-pin option collapses into this same overflow menu (shopify.dev/docs/apps/design/navigation).

Page titles and actions render through the `s-page` component's slots:

```
<s-page heading="Edit product">
  <s-badge slot="accessory" tone="warning">Draft</s-badge>
  <s-button slot="primary-action" onClick="...">Save product</s-button>
  <s-button slot="secondary-actions" commandFor="more-actions-id">More actions</s-button>
  <s-menu id="more-actions-id">
    <s-button icon="view">Preview</s-button>
    <s-button icon="export">Export</s-button>
  </s-menu>
  <s-link slot="breadcrumb-actions" href="/">Home</s-link>
</s-page>
```

Rules that go with this:

- Keep the page title short and scoped to a single purpose. Try to limit each page to one job; splitting merchant attention across multiple purposes on one screen damages usability more than a second page costs in navigation overhead.
- Only one primary action per page. This is also the non-negotiable this whole skill repeats elsewhere: never two CTAs with the same intent competing for the same click.
- Button labels lead with a strong verb in a `{verb} + {noun}` format ("Save product," "Export report") so merchants can predict what will happen before they click.
- Group related secondary actions into a single `s-menu` behind `commandFor`, rather than scattering five same-weight buttons across the header.
- Use the accessory slot (e.g. `s-badge`) to show page-level status like "Draft" when relevant, so status doesn't have to be re-explained in the body copy.
- In a Remix app, anchor (`<a>`) tags aren't supported for title bar breadcrumb/action links — use Remix's `<Link>` component so client-side routing and App Bridge stay in sync.

## Tabs vs view-switchers vs filters: the distinction that gets misused

This is the single most common navigation mistake in Register A, because Shopify's own admin visually overloads one UI shape ("a row of tab-like buttons") across three semantically different jobs. Treat these as three different components with three different accessibility contracts, not one component with three skins.

| Pattern | What it actually does | Correct semantics | What happens on click |
|---|---|---|---|
| **Tabs-as-primary-navigation** | Moves the merchant to a different page/route of your app | This is App nav's job. Don't build this with tabs at all. | Full navigation to a new URL |
| **Tabs-as-view-switcher** | Changes which content renders below, within one page, without changing what's above | ARIA tabs pattern: a `tablist` of `tab` elements each pointing at a `tabpanel`, `aria-selected` on the active tab, one panel visible at a time | Only the content *below* the tabs changes; the tabs never move position, reorder, or wrap to multiple lines |
| **Segmented/pill filter control** | Filters which subset of one list is shown (e.g. All / Active / Draft on an index table) | A single-select, exclusive-choice control — closer to a radiogroup or a toggle-button-group than to tabs, since there's exactly one panel (the table) being re-queried, not N separate panels | The same table re-renders with a different filter applied; nothing structurally "switches," so it shouldn't inherit `role="tab"`/`role="tabpanel"` semantics |

Why this matters:

- Shopify's own guidance is explicit that **tabs should be used sparingly, only for secondary navigation, only when the App nav isn't sufficient** (Built for Shopify 4.1.4 / shopify.dev/docs/apps/design/navigation). The hard behavioral rules that come with legitimate tab usage: clicking a tab must only change the content below it, never above; tabs must never wrap onto multiple lines; and navigating between tabs must not cause the tabs themselves to change position or move. If your "tabs" reorder, reflow, or affect page-level chrome above them, you've actually built navigation dressed as tabs, and it should be an App nav item or a route instead.
- The trap specific to Shopify apps: the admin's own convention for index-table filters (All / Unfulfilled / etc. on an Orders-style list) visually renders as a row of tab-like controls, which is exactly why an older version of Shopify's own guidance had to say "don't use tabs as primary navigation, since tabs are used in Shopify admin as filters on index tables." That single sentence is doing a lot of work: it confirms Shopify's own admin uses the tab *look* for a filter *job*. Copying that look for your index table is fine; copying the ARIA tabs pattern's *underlying semantics* (separate `tabpanel`s, `aria-selected` driving which panel is in the accessibility tree) for something that's actually one panel being filtered is the mistake. Model an index-table filter row as an exclusive-choice control over one region, not as a tablist over several.
- Practically: if there is exactly one content region and clicking a control just changes *what data* fills it (a filter), don't reach for `role="tab"`. If there are conceptually separate views of the same object rendered as separate panels (e.g. "Details / Activity / Notes" on a single order), the ARIA tabs pattern is the right fit, provided you also honor the never-reposition, never-wrap, content-only-changes-below rules above.
- Keyboard behavior differs accordingly: a proper tablist uses a single `Tab` stop into the group, with arrow keys moving between tabs (roving tabindex) and Enter/Space activating the tab under focus. A radiogroup-style filter control follows the same single-Tab-stop-plus-arrow-keys pattern for the same reason (it's an exclusive choice), which is part of why it's easy to visually confuse the two, but the panel/state model underneath is different and should be coded accordingly.
- No specific `s-*` tab or segmented-control component name appears in the current research corpus for this skill. Don't invent one. Before implementing either pattern, run the search-then-validate loop in `shopify-polaris-app-home` (or validate with `validate_component_codeblocks`) to confirm the current tag name and props, and cross-check `reference/component-patterns.md` for anything already confirmed there.

## Deep-linking and URL state

Every destination reachable from the App nav, a breadcrumb, or a tab-as-view-switcher should correspond to a real, shareable URL, not just client-side component state that resets on reload. This is what makes browser back/forward, refresh, and merchant-shared links behave correctly inside the admin iframe:

- Use the Navigation API when you need to trigger navigation from JavaScript rather than a link click (e.g. after a save completes) — don't fake navigation with local state changes alone.
- In a Remix app, use `<Link>` rather than raw anchors for in-app navigation so App Bridge and the router stay synchronized; raw `<a>` tags aren't supported for title bar breadcrumb/action links specifically.
- `s-link href` values are relative paths within the app (e.g. `/products`, `/settings`) and navigate without a full page reload while still updating the URL, which is what deep-linking depends on.
- For a view-switcher (tabs) or a filter control, reflect the active tab/filter in the URL (a path segment or query parameter) wherever practical, so a merchant can bookmark, share, or reload into the exact state they left. The research corpus doesn't prescribe a specific query-param convention for this, so treat it as a general craft practice to apply, not an asserted Shopify API rule — validate any Remix/React Router loader wiring against your framework's own docs.
- A filtered-to-zero-results state (a real query, no matches) is not the same as an empty state (nothing exists yet) — see this skill's state-completeness guidance. This distinction matters most exactly where URL state is involved: a merchant who deep-links into `?status=archived` and sees zero rows needs a "clear filters" affordance, not first-run onboarding copy.

## Breadcrumbs and back navigation

- Merchants should always be able to return to the previous page without relying on the browser's back button. Provide breadcrumbs or a Back button on every sub-page (shopify.dev/docs/apps/design/navigation). A sub-page shipped with no way back to its parent is an explicit rejection reason under Built for Shopify.
- Implement breadcrumbs via the `s-page` breadcrumb slot (`s-link slot="breadcrumb-actions" href="..."`), not a hand-rolled row of links styled to look similar. Keeping it in the slot is also what makes it render consistently with the rest of the header chrome.
- Breadcrumbs communicate hierarchy ("Products > Blue T-Shirt > Variants"), while a Back button communicates simple linear return ("go back one step"). Use breadcrumbs when a page has a real parent/child resource relationship worth naming; use a plain Back action when the relationship is just "the page you came from."

## Growing past one page: when to add a nav item, a tab, or a modal

As an app's surface area grows, resist defaulting every new feature to a new top-level App nav item:

- **New App nav item**: the feature is a distinct, frequently-visited destination merchants need to jump to directly (not just something opened in the course of another task).
- **Interstitial nav / hub page**: when you have more settings or feature pages than comfortably fit in the sidebar, build a single hub page that links out to the individual sub-pages, rather than cramming every one of them into the App nav. This is a named Compositions pattern ("Interstitial nav") specifically for this job, and it keeps the sidebar itself down to "the fewest possible categories."
- **Tab (view-switcher)**: the content is a different facet of the *same* object or task the merchant is already looking at, and doesn't need its own row in the App nav or its own breadcrumb trail.
- **Modal**: the interaction is small, contained, and doesn't need its own URL to be useful (a confirmation, a short single-field edit). Don't use a modal as a substitute for a real page just to avoid adding a nav entry; large forms belong on a dedicated page with proper save-bar and back-navigation support, not squeezed into a modal (see `reference/forms.md` for the five-input threshold and save-bar rules).

## Navigation inside Admin UI extensions

Admin UI extensions (blocks, actions, print actions, configuration surfaces) sit outside your main App Home navigation, and have their own constraints worth designing around:

- Each `[[extensions.targeting]]` entry maps exactly one target (e.g. `admin.product-details.block.render`, `admin.product-index.action.render`) to one module. There's no shared nav shell across extension targets the way there is across App Home pages.
- Merchants must manually add and pin Block extensions to a resource page before they see them; design onboarding copy that accounts for this (a block can't self-promote its own existence the way an App nav item can).
- Returning `null` from a Block extension collapses it rather than removing it from the page entirely; don't design around the assumption that a block can be made to fully disappear at runtime.
- Action extensions can be launched from inside a Block extension for more complex interactions (the `extension:` URL protocol targets an `extensionTarget`), which is the closest equivalent to "drilling into a sub-page" available inside a block's constrained surface.
- Admin link extensions exist specifically to route merchants from an admin page into a related, more complex workflow inside your own app (the example given in the source material is Shopify Flow's admin link extension sending merchants to a page to run an automation for an order). Use this pattern, rather than a raw external link, when a block or action needs to hand off to a full page of your app.

## Pre-ship checklist

- [ ] Every top-level destination is reachable from `s-app-nav`, using short noun labels, with the fewest categories that still make sense.
- [ ] The app name shown in the nav is under 20 characters and doesn't truncate; the App Store listing name (if different) is under 30.
- [ ] No nav item duplicates the homepage link; only one `s-link` carries `rel="home"` if used.
- [ ] Sub-pages highlight the correct parent nav item and provide a breadcrumb or Back button.
- [ ] Every "tabs" instance changes only the content below it, never reorders, never wraps, and is genuinely secondary navigation, not a substitute for an App nav item.
- [ ] Every index-table filter row is modeled as a single-select filter over one panel, not as an ARIA tablist with multiple panels.
- [ ] Every reachable view (nav destination, tab-as-view-switcher, active filter) has a corresponding URL that survives reload and is shareable.
- [ ] No custom in-app navigation menu exists alongside `s-app-nav`; no navigation lives in the page header.
- [ ] No primary workflow requires leaving the Shopify admin.

## Handoff

For exact `s-app-nav` / `s-link` / tab-component prop names and enum values, validate against the live schema through the `shopify-polaris-app-home` marketplace skill (if installed) or `mcp__shopify-dev-mcp__validate_component_codeblocks` before shipping; don't trust an unverified prop name from memory. For Admin UI extension target strings and TOML targeting config specifics, defer to `shopify-polaris-admin-extensions`. For Built for Shopify / App Store navigation-related rejection criteria beyond what's cited here, run a check through `shopify-app-store-review` (if installed).
