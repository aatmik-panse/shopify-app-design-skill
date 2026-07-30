# Accessibility

This file covers Register A: the embedded admin surface (App Home, Admin UI Extensions, blocks, actions). Everything here is checkable, not aspirational. WCAG 2.1 AA is not a nice-to-have for a Shopify app: it is a named pillar of the Built for Shopify design requirements and an explicit App Store review expectation (shopify.dev/docs/apps/launch/built-for-shopify/requirements#design). An app that fails contrast, keyboard, or screen-reader basics can be rejected or lose Built for Shopify status regardless of how polished it looks.

One source note before the checklists: Shopify's dedicated accessibility page (shopify.dev/docs/apps/build/accessibility) is written for storefront themes first and reused near-verbatim for apps, so some of its examples (product variants, cart/hamburger buttons, slideshows) are storefront-flavored. The underlying HTML/ARIA/WCAG mechanics on that page are universal and apply directly to an iframe-embedded Polaris app. Treat the storefront-specific examples as illustrative, not literal, for App Home and admin extensions.

## Contrast: hard numbers, not vibes

- Body text and interactive text: minimum 4.5:1 contrast against its background (WCAG 1.4.3, restated at shopify.dev/docs/apps/design/visual-design as "maintain a background-to-text contrast ratio of at least 4.5:1").
- Large text (roughly 18pt+/14pt-bold+): minimum 3:1.
- Test every text/background pairing with an actual contrast tool (WebAIM Contrast Checker is the one Shopify's own docs point to) — don't eyeball it.
- Polaris's "subdued" text tone is the single easiest way to fail this. Subdued text sits close to the AA line on Shopify's neutral palette; never use it for anything load-bearing (error text, required-field markers, the only copy on a metric card). Reserve subdued for genuinely secondary caption text and re-check its contrast on every background it appears on (a card, a banner, a table row).
- Font-size floors compound the contrast problem: Shopify sets 13px as the minimum for headings/body/interactive text and 12px for captions (shopify.dev/docs/apps/design/visual-design). Small text and marginal contrast together fail AA even when either alone would pass. Check them together.
- Cap prose line length for readability (roughly 60-75 characters per line). A paragraph that runs the full width of a wide `s-section` is technically legible and practically unscannable.

## Color roles are semantic, never load-bearing alone

Shopify's admin color roles are fixed and merchants read them as status, not decoration (shopify.dev/docs/apps/design/visual-design):

| Color | Meaning | Never use for |
|---|---|---|
| Green | Positive status / successful completed action | Enticing merchants, drawing attention for its own sake |
| Yellow | Paused/incomplete status, non-urgent attention | Announcements |
| Orange | In-progress/pending, needs merchant attention (strongest non-blocking role) | "Coming soon" / "under construction" messaging |
| Red | Impossible/blocked action or error only | Enticing merchants, anything non-critical |

The accessibility rule sits directly on top of this: never convey status through color alone (shopify.dev/docs/apps/build/accessibility, shopify.dev/docs/apps/design/user-experience/alerts). Every status signal needs a second channel:

- Pair color with text ("Failed to sync", not just a red dot).
- Pair color with an icon (warning triangle, check circle) in addition to text where the UI is dense (a table row, a compact badge).
- A `s-badge` carrying a `tone` is not sufficient labeling by itself if the badge text alone doesn't state the status in words a screen reader will read out.

## Keyboard navigation and focus management

- Every interactive element needs a visible, consistent focus indicator, whether the merchant arrived there by mouse or keyboard (shopify.dev/docs/apps/build/accessibility). Never suppress the browser's focus ring without replacing it with something equally visible.
- Nothing in the app should depend on `:hover` to become visible or usable. If a row action, tooltip, or menu only appears on mouse hover, it is invisible and unreachable for keyboard and screen-reader users.
- Tab and Shift+Tab must move through the page in a sane, linear order. Only use `tabindex="0"` or `tabindex="-1"`. Never use a positive `tabindex` and never use `autofocus` — both hijack focus order away from the merchant.
- No sudden context changes when an element receives focus (a field gaining focus should never trigger navigation or a modal, for example).
- Zoom must always work. Never rely on complex multi-touch gestures for something that also needs a single-tap/click equivalent.
- For any custom dropdown/menu you build outside the `s-menu` primitive: Enter or Space opens it while focus stays on the launcher, Tab moves focus into the first item once open, and Esc collapses it and returns focus to the launcher.
- For any `s-modal` (or hand-built dialog-equivalent): on open, focus moves to the element that labels the modal (its heading); keyboard navigation is trapped inside the modal while it's open; Esc closes it and returns focus to whatever launched it; the role that identifies it to assistive tech is `dialog`. Verify the current `s-modal` implementation actually gives you this focus-trap behavior for free before assuming it does — check against shopify-dev docs or `validate_component_codeblocks`, don't assume.
- If you build a "skip to main content" pattern for a genuinely long custom App Home page (uncommon, since App Bridge chrome already handles most repeated navigation), make the skip link visible on focus and target `tabindex="-1"` on the content container so it's actually reachable.

## Screen-reader labeling for s-* components

Every icon-only control needs an explicit accessible name. Visible icon glyphs are not read aloud; a `s-button` with only an `icon` prop and no visible text is silent to a screen reader unless labeled.

```html
<s-button icon="x" variant="tertiary" tone="neutral" accessibilityLabel="Dismiss setup guide"></s-button>

<s-spinner size="base" accessibilityLabel="Loading recent orders"></s-spinner>

<s-button commandFor="row-actions-42" icon="menu-vertical" accessibilityLabel="More actions for order #1042"></s-button>
<s-menu id="row-actions-42" accessibilityLabel="Order #1042 actions">
  <s-button icon="view">Preview</s-button>
  <s-button icon="export">Export</s-button>
</s-menu>
```

Concrete rules:

- `s-spinner` has no text of its own by definition; always pair it with `accessibilityLabel` describing what is loading, not just "loading" repeated everywhere ("Loading recent orders" beats "Loading").
- Any icon-only `s-button` (dismiss, overflow trigger, row action) needs `accessibilityLabel`. If the button also has visible text, it doesn't need one; don't double-label.
- `s-menu` and the trigger that opens it (`commandFor`) should both be identifiable: the trigger via `accessibilityLabel` if icon-only, the menu itself via `accessibilityLabel` describing its contents ("Order actions", not "Menu").
- `s-chip` / `s-clickable-chip` need `accessibilityLabel` when their visible content is abbreviated, an icon, or ambiguous out of context (a filter chip showing just a count, for example).
- Search-style fields that visually hide their label (a compact search box with a placeholder standing in for the label) still need a programmatic label reaching screen readers; check the current field component's props for the mechanism (something like an accessibility-visibility prop) rather than assuming placeholder text alone is sufficient. Placeholder text is not a substitute for a label under WCAG.
- `s-avatar`, `s-image`, `s-thumbnail` all need descriptive `alt` text when they carry meaning (a customer's photo, a product image). Purely decorative imagery uses empty `alt=""` so screen readers skip it (shopify.dev/docs/apps/build/accessibility).
- Never put a screen-reader label on a container and assume it "covers" every interactive child inside it. Label each control individually.
- Cross-check every `accessibilityLabel`-bearing prop name against the current component API before shipping. Prop-name drift between API versions is exactly the kind of thing `validate_component_codeblocks` (mcp__shopify-dev-mcp) exists to catch. For the exhaustive per-component prop reference, hand off to the `shopify-polaris-app-home` or `shopify-polaris-admin-extensions` marketplace skills (if installed) rather than trusting a remembered prop name here.

## Dynamic content and live regions

Anything that changes on screen without a full navigation (a status flipping from "Queued" to "Failed," a save confirming, a count updating after a bulk action) needs to reach assistive tech, not just sighted users watching the pixels change.

- Route routine, non-urgent updates (a save completed, a count refreshed) through a polite live region so they're announced without interrupting whatever the merchant is doing.
- Reserve assertive announcements for something that actually blocks the merchant's task (a failed action that needs attention right now).
- If you're relying on `shopify.toast.show()` or a built-in `s-banner` appearing dynamically, verify it actually gets announced to screen readers by the current runtime rather than assuming it does. If you hand-roll any status text update outside the component system, wrap it in an explicit live region yourself.
- A banner that only communicates state through a background-color change (no new text, no icon) is invisible to this whole mechanism no matter how you wire the live region. Fix the content first (see the color-roles table above), then make sure the content is announced.

## Motion and prefers-reduced-motion

Tasteful custom motion (a chart drawing in, a skeleton shimmer, an onboarding sequence stepping through) is one of the few legitimate creative frontiers inside Register A, but every instance of it is conditional on respecting the merchant's motion preference.

```css
@media (prefers-reduced-motion: reduce) {
  .your-custom-chart-enter,
  .your-custom-skeleton-shimmer,
  .your-custom-onboarding-step {
    animation: none !important;
    transition: none !important;
  }
}
```

- Any animation you author yourself (not a built-in component's internal transition) must have a reduced-motion path that removes or drastically shortens it, never one that just "goes a bit faster."
- This applies to media controls specifically per Shopify's own guidance (shopify.dev/docs/apps/build/accessibility): don't force autoplay or motion on merchants who've opted out.
- A loading skeleton's shimmer animation is real motion. Gate it the same as anything else, and make sure the skeleton's *shape* still matches the loaded layout even with the shimmer disabled, since shape-matching is the actual point of a skeleton, not the shimmer.

## Forms

- Every field needs a label reaching assistive tech, whether that's a visible label, an `aria-label`-equivalent prop, or a visually-hidden label element. A placeholder alone is never a label (shopify.dev/docs/apps/build/accessibility).
- Mark required fields with the actual required semantics the field component exposes, not just an asterisk in the visible label text.
- Use autocomplete hints on fields where a sensible one exists (email, URL, password) so browsers and assistive tech can help merchants fill them faster.
- On validation error: move focus to the error feedback (the field or a linked summary), write a clear and specific message ("Enter a valid email address," not "Invalid input"), and link the field to its error text programmatically so a screen reader announces them together when the field is focused.
- Never validate while the merchant is still typing. Wait until focus leaves the field before showing an error — mid-keystroke validation reads as the UI attacking the merchant's input (shopify.dev/docs/apps/design/user-experience/alerts).
- Error text must be both colored (red, per the color-roles table) and textual/iconified. Never red-highlight a field with no accompanying message.
- Errors that apply to a whole section belong near the top of that section, not buried three components deep, and definitely not surfaced only inside a modal that isn't itself the thing erroring.

## Tables

- If you're hand-building tabular markup instead of using the table component, use real `<table>`/`<caption>`/`<th scope="col">`/`<th scope="row">` semantics so screen readers can navigate by row and column header, not just visually align text (shopify.dev/docs/apps/build/accessibility).
- Prefer the built-in table component for anything resource-list-shaped; it's built to carry header/row semantics correctly. Confirm its current header-slot props actually produce accessible column headers rather than assuming, and cross-check against `reference/component-patterns.md` for the full prop surface.
- Row actions in a table should be reachable by keyboard and individually labeled (see the `s-menu`/`commandFor` example above) — a hover-only action column fails the "no mouse-hover dependency" rule twice over: once for keyboard, once for touch.

## Touch targets

- Primary interactive controls need at minimum a 44x44px hit target (shopify.dev/docs/apps/build/accessibility). This matters most for icon-only buttons, row actions, modal close buttons, and anything rendered inside a POS UI extension or on a narrow admin viewport, where visual size is often shrunk to save space but the tap target still needs to stay reachable.
- Provenance caveat: 44x44px is the one concrete touch-target number Shopify publishes, but it comes from Shopify's single cross-surface accessibility doc, which is written largely for storefront themes (its worked examples are cart/hamburger buttons, product variant controls, slideshows) and reused near-verbatim for apps — it isn't authored specifically for App Home or admin UI extensions. Treat 44x44px as the right ballpark, not a verified admin-specific spec, and reconfirm it for App Home / admin-extension surfaces before treating it as gospel (check current shopify-dev guidance, or hand off to `validate_component_codeblocks` / the `shopify-polaris-app-home` / `shopify-polaris-admin-extensions` skills if installed).
- Don't shrink a button's visual footprint below 44x44px just because its icon looks fine smaller. Pad the hit area even if the icon itself is compact.

## Reading this against Built for Shopify and App Store review

WCAG 2.1 AA contrast is explicitly named as part of the "Accessible" pillar in Shopify's App Design Guidelines and is called out again under the Built for Shopify design requirements (shopify.dev/docs/apps/launch/built-for-shopify/requirements#design). Concretely, reviewers and automated checks will catch:

- Text that fails 4.5:1 (or 3:1 for large text).
- Status conveyed by color alone.
- Missing back button / broken keyboard trap on a sub-page or modal.
- Icon-only controls with no accessible name.
- Motion that ignores `prefers-reduced-motion`.

For a full pre-submission compliance pass (not just accessibility), hand off to the `shopify-app-store-review` skill if installed. For validating that the exact `s-*` markup you wrote compiles against the current component schemas (including the accessibility-relevant props named above), run it through `validate_component_codeblocks` (mcp__shopify-dev-mcp) before calling anything shipped.

## Pre-ship accessibility checklist

- [ ] Every text/background pairing checked at 4.5:1 (3:1 for large text), including subdued text on every background it appears on.
- [ ] No status communicated by color alone anywhere in the page (badges, banners, table rows all paired with text/icon).
- [ ] Unplug the mouse: complete the page's primary workflow using only Tab, Shift+Tab, Enter, Space, and Esc.
- [ ] Every icon-only button, menu trigger, and chip has an `accessibilityLabel` (or equivalent) that names what it does, not just "button" or "menu."
- [ ] Every `s-spinner` has an `accessibilityLabel` describing what's loading.
- [ ] Modals/drawers: focus moves to the heading on open, Esc closes and returns focus to the launcher, focus can't escape the modal while open.
- [ ] No `autofocus`, no `tabindex` values other than `0`/`-1` anywhere.
- [ ] Every custom animation has a `prefers-reduced-motion` path that removes or shortens it.
- [ ] Form errors: focus moves to the error, message is specific, error text is red AND textual, validation doesn't fire mid-keystroke.
- [ ] Every image/thumbnail/avatar has descriptive `alt`, or empty `alt=""` if purely decorative.
- [ ] Touch targets on icon-only and row-action controls measure at least 44x44px (reconfirmed for admin/App Home context, not just assumed from the storefront-flavored source doc).
- [ ] Spot-checked with a real screen reader (VoiceOver or NVDA), not just visually reviewed.
- [ ] Ran the actual `s-*` markup through `validate_component_codeblocks` before calling it done.
