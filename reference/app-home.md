# App Home

App Home is the app's own territory inside Shopify Admin: the landing page merchants see when they click your app's name in the sidebar, plus whatever settings and sub-pages hang off it. Two genuinely different technical surfaces both answer to the name "App Home." Get the distinction right before you design anything, because it changes what you can build.

## The two App Home surfaces

### App Home (iframe) — the standard surface

This is what most Shopify apps mean by "App Home." Your app runs its own backend, and Shopify Admin loads your app's pages inside an iframe. Two things are true at once, and they're easy to conflate:

- **App Bridge** (the JavaScript SDK) controls the Shopify Admin chrome *around* your iframe: page title, nav menu, save bar behavior. This is `TitleBar`, `NavMenu`, `ui-save-bar`, and the `useAppBridge` hook from `@shopify/app-bridge-react`.
- **Polaris web components** (the `s-*` custom elements: `s-page`, `s-section`, `s-button`, `s-box`, and the rest) build the actual UI *inside* your iframe. They're globally registered custom elements — no import statement, usable from React, Vue, or vanilla JS (shopify.dev/docs/api/app-home).

Auth is automatic. The app runs inside an authenticated admin session; backend calls use a session token via `shopify.idToken()` sent as an `Authorization: Bearer` header, and direct frontend calls to `shopify:admin/api/graphql.json` are auto-authenticated by App Bridge. You still have to declare access scopes in `shopify.app.toml`'s `[access_scopes]` before you can query them, and if you want the frontend to call the Admin GraphQL API directly (rather than proxying through your own backend), you must explicitly opt in with `embedded_app_direct_api_access = true` under `[access.admin]` and set `direct_api_mode` to `"online"` or `"offline"` (shopify.dev/docs/api/app-home).

Because this surface has a real backend, it's the only option when your app needs webhooks, background jobs, other server-side functionality, or server-side rendering (shopify.dev/docs/api/app-home-ui-extension/latest). The homepage route conventionally lives at `app._index.jsx` in React Router/Remix-scaffolded apps (shopify.dev/docs/apps/design/user-experience/app-home-page).

### App Home UI Extension — a distinct, narrower target

This is a different extension target, `admin.app.home.render`, and it is genuinely not the same thing as iframe App Home wearing a different hat. Confirmed specifics from the extension reference (shopify.dev/docs/api/app-home-ui-extension/latest):

- **No separate web server, no iframe, no server-side rendering.** Your extension bundle is rendered by Shopify Admin directly, via remote-dom.
- **Maximum one `admin.app.home.render` target per app.** You don't get multiple home surfaces this way.
- **Custom-distribution apps only.** This is explicitly not for apps distributed through the public Shopify App Store.
- **Choose iframe App Home instead if you need webhooks, background jobs, server-side functionality, or server-side rendering** — the extension has none of those.
- **Hard size limits enforced at `shopify app deploy` time:** compiled bundle capped at 64 KB compressed; static assets capped at 1 MB per file and 50 MB / 500 files total for the whole asset bundle. SVG files are explicitly not supported as assets.
- The UI is built from the same Polaris `s-*` vocabulary as iframe App Home, and the same global-object pattern applies: `shopify.query()` for GraphQL, `shopify.intents`, `shopify.toast.show()`, `shopify.storage`.

**What the research doesn't resolve:** the full comparison doc between the two ("Apps in App Home") was only partially retrieved. The one clear, confirmed cutover rule is the webhooks/background-jobs/server-side/SSR test above. Whether the Homepage-page UX guidance below (setup guide, metrics card, etc.) applies identically to the UI Extension surface, or whether it has its own separate UX guidance, is not confirmed in this research — treat the composition guidance in this file as written for the iframe-based approach, and verify against current docs (or the `shopify-polaris-app-home` marketplace skill) before assuming parity for a UI Extension build.

### Decision quick-reference

| If your app needs... | Use |
|---|---|
| Webhooks, background jobs, or its own backend logic | App Home (iframe) |
| Server-side rendering | App Home (iframe) |
| Public App Store distribution | App Home (iframe) — the UI Extension target is custom-distribution only |
| A lightweight, single-purpose home surface for a custom/private app with no backend | App Home UI Extension |

For the exact `s-*` tag syntax, prop casing rules, and the boolean-vs-string-attribute gotchas that apply identically to both surfaces, see reference/component-patterns.md rather than re-deriving them here.

## The App Home page itself

Whichever surface you're building on, the home page's job is narrow and specific: "provide status updates, enable merchants to respond to immediate needs, and provide clear CTA buttons so that merchants can easily see the actions they can perform in the app" (shopify.dev/docs/apps/design/user-experience/app-home-page). It's the first thing merchants see on every visit, and its stated use cases are exactly three: a dashboard of key metrics and status, surfacing items that need attention, and guiding new users through onboarding.

Everything below follows from taking that job description literally. A home page that doesn't do one of these three things for a specific merchant, right now, is decoration.

### The framing: state, then priority, then next action

Before placing anything on the home page, work through this in order, not the reverse:

1. **What state is this merchant actually in?** New install with no data yet. Installed and working normally. Something needs their attention (an error, an expiring connection, an exception). These are mutually exclusive top-level states, and the page should look different in each one.
2. **Given that state, what is the single most important thing they need to know right now?** Not everything you could show them, the one thing.
3. **What is the one primary next action tied to that?** One dominant action, derived from the actual state, not a menu of equally-weighted options.

Concretely:

| Merchant state | Home page should lead with | Home page should NOT lead with |
|---|---|---|
| New install, no data yet | Setup guide with one clear next step, or an empty state with a single CTA | A dashboard of metrics cards showing all zeroes; a marketing/welcome hero |
| Working normally, healthy | A metrics summary tied to a decision the merchant might make, plus whatever's next in their workflow | A wall of equal-weight KPI cards with no hierarchy; vanity numbers with no action attached |
| Needs attention (error, warning, expiring connection) | A banner or inline alert near the top, with the fix as the primary action | A callout card promoting a new feature; congratulatory copy that ignores the exception |

This is also where the single most common home-page mistake shows up: a flat grid of same-size KPI cards with no hierarchy tied to what the merchant should actually do next reads as a generic SaaS dashboard, not a Shopify admin surface, and it fails the state-priority-action test by definition (no single card is "the" priority). A returning user should never land on a marketing hero or welcome banner sitting where the live state and next action belong — that's the second most common failure, and it directly contradicts the page's stated job of surfacing status and immediate needs. Any number shown on the home page needs a decision attached to it; if a merchant can't do anything differently after seeing it, it hasn't earned its place above the fold.

### Composition patterns for the home page

These are the named, pre-built pattern categories the docs point you toward for the home page rather than freehanding a layout (shopify.dev/docs/api/app-home, shopify.dev/docs/api/app-home-ui-extension/latest). Use them as starting points; validate exact markup with `validate_component_codeblocks` before shipping (see reference/component-patterns.md for tag-level syntax).

**Setup guide.** An interactive checklist with visual progress tracking for onboarding, meant to live on the home page or a dedicated onboarding page. Keep instructions brief and direct; only ask for information you actually need. Mark each task complete the moment the merchant finishes it, and show a progress readout ("0 out of 3 steps completed"). Persist expanded/dismissed state so it doesn't reset on every visit. Cap the process at five steps or fewer, because additional steps measurably increase drop-off; if the onboarding isn't essential, make it dismissible via the Polaris Cancel icon rather than forcing every merchant through it (shopify.dev/docs/apps/design/user-experience/onboarding, shopify.dev/docs/apps/design/user-experience/app-home-page).

**Empty state (`s-empty-state`).** For a blank page, list, or table: a heading, an optional graphic, a subheading, and one prominent action that says what will appear here (shopify.dev/docs/api/app-home/latest/web-components/feedback-and-status-indicators/empty-state). Outside a table, render it only when the resource is empty. Inside `s-table-body` it replaces the rows. A filtered-to-zero result can use the same component, but the copy is "clear filters," not onboarding.

**Metrics card.** Surfaces key numbers, statistics, or trends so a merchant can read activity or performance at a glance. Include a trend indicator only when a historical comparison actually adds context, not by default. Because this is the component most prone to becoming a vanity-metric wall, hold every metrics card to the same bar: does this number change what the merchant does next? If the underlying data fetch fails, the card needs a banner with an inline Retry, not a silent fallback to a blank or zeroed card.

**Callout card.** Use sparingly, to announce a new feature, promote functionality, or guide a merchant through a common first action. One clear message, one prominent button. Remove the callout entirely once the merchant has engaged with it — a callout that keeps reappearing after being acted on stops reading as help and starts reading as nagging. Never let a callout compete with the page's actual primary action; a home page should still have exactly one dominant CTA.

**Account connection.** Named in the composition catalog for showing whether a linked third-party account or service is connected, with the obvious connect/disconnect actions. The research corpus names this composition but does not detail its internal layout or props beyond that — verify the current anatomy via the `shopify-polaris-app-home` marketplace skill or `validate_component_codeblocks` before building against it, rather than assuming a specific structure here.

**Footer help.** Support and documentation links, placed at the bottom of the page or section they relate to, with descriptive link text that tells the merchant exactly where they're going (documentation, support, legal). Support needs to be consistently discoverable without getting in the way — an app nav item, a footer link, or a floating action button are all suitable placements. One confirmed code shape:

```
<s-stack alignItems="center">
  <s-text>Learn more about <s-link href="">creating puzzles</s-link>.</s-text>
</s-stack>
```

**Banners.** Only one banner should be visible on the page at a time. Persist dismissal (local storage or your database) so a dismissed banner doesn't resurface for that merchant. This is the general alert-placement rule from reference material on alerts, restated here because the home page is where banner pile-up is most tempting and most damaging to trust.

**Other named compositions worth knowing exist**, even though they're less central to the home page specifically: **App card** (a tappable card recommending a complementary or partner app), **Interstitial nav** (a hub page that links out to deeper settings or feature pages without cluttering the main nav), **Media card** (visual information paired with actionable, educational content), **Index table** and **Resource list** (scannable collections, more relevant when "items needing attention" surfaces on the home page as an actual list rather than a summary count). All are named in the same Compositions catalog; several were not detailed beyond their name and one-line purpose in this research, so treat their exact anatomy as unconfirmed until checked against current docs.

### The documented reference shape (read as illustrative, not mandatory)

One of the pattern-template walkthroughs shows a homepage composed, in order, as: a dismissible banner, a collapsible Setup Guide, Metrics cards, a Callout card, a template grid, a news section, and a featured-apps section, with each dismissible piece persisting its own state (shopify.dev/docs/apps/design/user-experience/app-home-page). Treat this as one worked example, not a template every app must reproduce section-for-section — the template grid, news section, and featured-apps section in particular look specific to the sample app used in that walkthrough. What generalizes is the underlying discipline: dismissible sections persist their state, only one banner shows at a time, and the state-priority-action framing above decides which of these sections a given app actually needs, and in what order, for its own merchants.

### Home page checklist

Check the universal non-negotiables in SKILL.md first (one dominant primary action, empty-state vs. filtered-to-zero distinction, dismissal persistence, skeleton-matches-loaded-shape, translation-length safety) — they apply here too and aren't re-derived below. What's specific to this page:

- [ ] One-line design read done first: what state is this merchant in, what's the priority, what's the one next action.
- [ ] Setup guide (if present) is five steps or fewer, dismissible if non-essential, and shows real progress.
- [ ] Every metrics card ties to a decision; none are there purely for visual balance.
- [ ] Metrics/data fetch failures show a banner with inline Retry, never a silent blank state.
- [ ] Callout cards are used sparingly and removed once acted on.
- [ ] Returning users with working setups do not see a marketing hero or welcome banner where live status belongs.
