# Admin UI extensions: choosing the right contextual surface

This file owns one question: **when should a piece of functionality live in an Admin UI extension instead of the app's own page, and which extension type is the right placement.** It does not own component props, target-string syntax, or the codegen/validate loop — that belongs to the `shopify-polaris-admin-extensions` skill (if installed) and the `validate_component_codeblocks` MCP tool. For the App Home / iframe surface (the app's own pages), see reference/app-home.md. For the `s-*` gotcha list and prop syntax rules, see reference/component-patterns.md.

## The core judgment call

An Admin UI extension is the right choice whenever a merchant already has a workflow open (a product page, an order, a bulk selection, a print job) and your app's job is to **add one contained capability to that workflow** rather than to be the destination itself. The wrong instinct is to build everything as a full app page and rely on merchants to remember to visit it. The other wrong instinct is to cram something that genuinely needs its own workspace (a multi-step form, a dashboard, a long report) into a 64 KB extension bundle that renders inside someone else's page.

Ask the placement question in this order:

1. Does the merchant need to see this or act on it *right now, in context*, without leaving the page they're already on? → extension.
2. Does it need real workspace (multiple sections, a big form, a chart, a multi-step flow)? → the app's own page (App Home), and use a Link extension only as the doorway to get there.
3. Is it a document to preview and print? → Print action, nothing else.
4. Is it a bulk operation over merchant-selected resources? → Action extension targeting the selection-action variant.

All four extension types run inside Shopify's chrome, not your app's iframe. They inherit Register A rules in full: Polaris `s-*` components only, no custom fonts, no brand-color takeover (shopify.dev/docs/apps/design/visual-design). The single most common Admin UI extension mistake is trying to make the extension feel like "our app" rather than "a native Shopify capability that happens to be provided by our app."

## Extension types at a glance

Shopify's own admin-extensions API docs enumerate six canonical target types: Action, Selection action, Block, Configuration, Print action, and Runnable (shopify.dev/docs/api/admin-extensions/latest). This file folds Selection action into the Action section below, since it shares the same Render / Should-render mechanism and modal-based interaction model — see "The selection-action variant" under Action extensions. Configuration and Runnable are handled in the "Two more specialized targets" section further down, since they're narrow and feature-specific rather than general creative-placement decisions a design pass needs to make.

Link is not one of the six canonical target types. It's a distinct, separately CLI-scaffolded pattern (`admin_link` template, not `admin_action`) that has no rendered content of its own but surfaces in the same "More actions" menu as Action extensions, as a pure navigation bridge. It's included in the table below as a practical fifth option, alongside the target-type families this file covers in depth, because "a contained modal here" (Action) versus "a doorway to a full page" (Link) is a real, common design decision this file needs to resolve — even though Shopify's own taxonomy doesn't group it with the six.

| Type | Lives in | Real estate | Interaction model | Reach for this when |
|---|---|---|---|---|
| Action | "More actions" menu on a details or index page | A modal, opened on demand | Render (menu item + modal content) and Should render (visibility condition) | A merchant needs to perform one focused task on the resource they're already looking at, without navigating away |
| Block | Inline on a resource page (product, order, customer details) | A card-sized slot the merchant has pinned into their page layout | Always rendered when pinned; can launch an Action extension for anything more complex | A merchant benefits from seeing your app's data or status *every time* they look at this resource, with no click required |
| Link | "More actions" menu, but the entry navigates into your app | None — it has no rendered surface of its own | Pure navigation bridge, not a canvas | The task is too big for a modal but the merchant discovered the need for it from this specific resource |
| Print action | The Print menu (not More actions) on order and product pages | A document preview | Preview-and-print flow via a dedicated print API | The output is a physical or PDF document (packing slip, certificate, custom label), not an on-screen result |

(Target-type list and per-type descriptions: shopify.dev/docs/api/admin-extensions/latest)

## 1. Action extensions

**What it is:** an entry in the "More actions" menu on a resource details page or an index page. Selecting it opens your extension's content in a modal (shopify.dev/docs/api/admin-extensions/latest, "Add menu items to the More actions menu on details and index pages").

**Two render varieties, always paired:**
- Render — the menu item itself, plus what shows in the modal once clicked.
- Should render — a non-visual check that decides whether the menu item appears at all for the current resource/state. Use this to keep the menu clean: if your action only makes sense for orders with a certain fulfillment status, or products missing a certain field, hide it otherwise rather than showing it disabled.

**The selection-action variant:** when merchants multi-select resources on an index page and open "More actions," a distinct target surfaces menu items scoped to bulk operations — "batch exports, bulk tagging, or multi-item processing" is the explicit recommended use case in the source docs (shopify.dev/docs/api/admin-extensions/latest). It shares the same Render / Should-render mechanism as a single-resource Action and targets the index page's selection context rather than the single-resource context. The exact CLI scaffolding path for selection-action (whether it reuses the `admin_action` template or has its own) isn't independently confirmed in the source docs — verify both the generation command and the exact target string against current docs, or hand this off to the `shopify-polaris-admin-extensions` skill, before writing the TOML.

**Design constraints that actually matter here:**
- You are in a modal, not a page. Anything you'd normally spread across a settings page with sections and a save bar does not fit. The Forms guidance is explicit and transfers directly: avoid placing large forms inside max-height, max-width modals — create a full app page instead (shopify.dev/docs/apps/design/user-experience/forms). If your action needs more than a handful of inputs, that is a signal you picked the wrong extension type, not a signal to make the modal bigger.
- One task per action. A modal that tries to do three things is a workflow you should split into three separate menu entries, each individually Should-render-gated.
- The modal needs its own primary action, its own loading state while it works, and its own success/error feedback before closing — it is a self-contained mini-flow, not a preview of something that finishes elsewhere.

**Right creative placement example:** "Regenerate SEO description" on a single product's More actions menu (single-resource Action); "Bulk-tag with AI-detected category" on the product index after selecting 40 rows (selection-action). Wrong placement: a full onboarding wizard crammed into an Action modal — that belongs on your App Home page, reached via a Link extension instead.

## 2. Block extensions

**What it is:** an inline card rendered directly on a resource page — product details, order details, customer details (shopify.dev/docs/api/admin-extensions/latest, "Render inline cards on resource pages like product details, order details, or customer details").

**The design constraint merchants impose on you, not the other way around:** merchants must manually add and pin a block to their page before it ever appears (same source). This changes the design brief completely. A block extension has to earn its spot in a "More actions" style discovery moment first (usually you point merchants at "Customize this page" and explain what your block adds), and then, once pinned, it has to justify the vertical real estate it permanently occupies every time that page loads. A block that shows a static label or a vanity number that never changes what the merchant does next is exactly the "flat KPI card with no attached decision" anti-pattern — it will get unpinned.

**Block real estate is small and fixed.** Design it as a card, not a page:
- Lead with the one thing the merchant needs from your app about *this specific resource*, right now (a status, a score, a warning, a next action) — not a menu of everything your app can do.
- If the interaction needs to grow beyond "glance and maybe click one button," launch an Action extension from inside the block for the deeper flow, rather than trying to expand the block itself (source docs explicitly note Action targets can be launched from blocks for exactly this reason).
- Design the collapsed/summary state deliberately — blocks commonly expose a collapsed-summary affordance so merchants can keep the card small until they need the detail (`<s-admin-block heading="Custom Fields" collapsedSummary="3 fields configured">…</s-admin-block>` is the representative pattern from the source docs; verify exact prop names against current docs before shipping).

**A hard runtime constraint to design around:** returning null from a block extension collapses the block — it does not remove it from the page (shopify.dev/docs/api/admin-extensions/latest). You cannot fully hide a pinned block at runtime the way you can hide an Action menu item with Should-render. Design the collapsed state to still look intentional (a one-line "nothing to show for this product yet" rather than an empty box), because merchants will see it.

**Right creative placement example:** an inline "AI visibility score: 62/100 — 2 fixes available" card on the product details page that expands to a short list and launches a full audit via an Action extension for the deep fix flow. Wrong placement: reproducing your entire dashboard inside the block — that is what App Home is for.

## 3. Link extensions

**What it is:** a "More actions" menu entry, like an Action extension, but it has no rendered content of its own — selecting it navigates the merchant into a page inside your app (shopify.dev/docs/api/admin-extensions/latest names Admin link extensions as directing merchants "to related complex workflows inside the app itself," citing Shopify Flow's link extension that sends merchants to a page to run an automation for an order).

**This is the escape valve for "too big for a modal, but the merchant found the need for it here."** Use a Link extension exactly when you'd otherwise be tempted to overstuff an Action modal: the task genuinely needs a full page (a multi-step configuration, a report, a form with more than a handful of fields), but the entry point that makes sense to the merchant is this specific resource, not your app's own nav.

**Design constraint:** because a Link extension has no surface of its own, the entire design burden shifts to (a) the menu label being unambiguous about where the merchant is about to go and what will happen, and (b) the destination page inside your app landing the merchant in exactly the right, pre-scoped context (the order/product/customer they came from should already be selected or pre-filled on arrival, never a blank page asking them to search for what they just clicked from). A Link extension that dumps merchants on your generic homepage after they clicked "Run automation for this order" has failed the one job it exists to do.

**Right creative placement example:** "Set up store-language rules for this product" opens the app's full rules-builder page with this exact product pre-loaded. Wrong placement: using a Link extension for something that's actually one button and one confirmation — that belongs in an Action modal instead, since forcing a full page load and navigation for a two-second task adds friction, not clarity.

## 4. Print action extensions

**What it is:** an entry in the Print menu — explicitly not the More actions menu — on order and product pages, for generating a custom document (shopify.dev/docs/api/admin-extensions/latest, "Add menu items to the Print menu on order and product pages"). It ships with dedicated APIs to preview and print the document, and has the same Render / Should-render pairing as Action extensions (verified target names: `admin.product-details.print-action.render` and `admin.product-details.print-action.should-render`).

**Design constraint:** the merchant's mental model here is "generate a document," not "run a feature." Keep the surface to a preview of the actual document plus a print action — resist adding unrelated settings or promotional content into this flow, since the merchant is mid-task on a genuinely narrow job (get a packing slip, a certificate, a custom label onto paper or PDF). Use Should-render to keep the Print menu clean: only show your entry when the current order/product actually has something printable (e.g., don't show a "print certificate of authenticity" action on an order with no eligible line items).

**Right creative placement example:** a custom packing slip with your app's compliance stamp, offered from an order's Print menu. Wrong placement: anything that isn't ultimately a document to print — that always belongs in an Action or Block extension instead.

## Two more specialized targets exist outside this file's scope

Configuration targets (for bundle/customizable-product setup, and for Shopify Function settings like discount/validation/order-routing) and Runnable targets (headless, no UI, used for things like Sidekick data providers) are real target types in the admin-extensions API but are narrow, feature-specific surfaces rather than general-purpose creative placements a design pass needs to choose between. Together with Action, Selection action (folded into the Action section above), and Block, these complete Shopify's six canonical admin-extension target types (shopify.dev/docs/api/admin-extensions/latest); Link, covered as this file's third section, sits outside that six-item taxonomy entirely as a related, separately-scaffolded navigation pattern. Treat Configuration and Runnable as implementation detail — hand them to the `shopify-polaris-admin-extensions` skill or `search_docs_chunks` when the feature actually calls for one (shopify.dev/docs/api/admin-extensions/latest).

## Constraints that apply across all four types

- **Performance is not optional here.** "Extensions run in the context of key merchant workflows, so always prioritize performance" is stated as a top-level principle, not a nice-to-have (shopify.dev/docs/api/admin-extensions/latest). A slow block extension delays the entire resource page it's pinned to.
- **Compiled bundle can't exceed 64 KB.** This alone rules out shipping a rich dashboard or a heavy chart library inside any of these four surfaces — that content belongs on your App Home page instead.
- **One target per module.** Each `[[extensions.targeting]]` entry in the TOML maps exactly one target to one file. Don't try to make a single module conditionally behave as both a Block and an Action — scaffold them as separate extensions.
- **Component model is version-locked, not gradual.** For API version 2025-07, extensions use React components from `@shopify/ui-extensions-react/admin` only. For every other version, they use Polaris `s-*` web components only. This is a hard either/or split per the source docs — confirm which API version the project targets before writing markup, since the two are not interchangeable.
- **Always scaffold with the CLI.** `shopify app generate extension --template admin_action|admin_block|admin_link|admin_print --name <name>` — the source docs state this as an imperative ("ALWAYS USE THE CLI TO SCAFFOLD A NEW EXTENSION") because hand-built templates drift from the current target/version contract.
- **No HTML comments in extension code.** The validator treats `<!-- ... -->` as an invalid custom component — a real, tooling-confirmed gotcha, not a style preference.

## Anti-patterns specific to extensions

| Don't | Do instead |
|---|---|
| Build a settings form with 8+ fields inside an Action modal | Use a Link extension to send the merchant to a full settings page in your app |
| Leave a pinned Block showing a static, never-changing value | Make every value in a block tied to something the merchant might act on, or don't pin it |
| Assume returning `null` hides a Block | Design an intentional collapsed/empty state; a block can't be fully hidden at runtime |
| Land merchants on your generic homepage after a Link extension click | Pre-scope the destination page to the exact resource the merchant came from |
| Add promotional or upsell content inside a Print action's document preview | Keep Print action flows narrowly about the document being generated |
| Show a menu item (Action, selection-action, or Print action) that doesn't apply to the current resource | Gate it with the Should-render variant instead of showing a disabled or irrelevant entry |
| Try to make an extension "look like our brand" with custom color/typography | Extensions are Register A — Polaris tokens and `s-*` components are the visual truth here, same as App Home (see reference/app-home.md) |

## Handoff

Once you've picked the right extension type and design intent for it, hand off to the `shopify-polaris-admin-extensions` skill (if installed) for the actual target-string, TOML config, and component-prop implementation — it runs the mandatory search-then-validate loop against real component schemas. If that skill isn't installed, use the `mcp__shopify-dev-mcp` tools directly: `search_docs_chunks` and `learn_shopify_api` for the current target/component reference, and `validate_component_codeblocks` on any `s-*` markup before it ships. For Built for Shopify / App Store compliance review of the finished extension, use the `shopify-app-store-review` skill (if installed).
