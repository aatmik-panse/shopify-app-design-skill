# Alerts and feedback

Scope: Register A (embedded admin surface) only. This file covers how to choose, tone, dismiss, and persist the feedback patterns Shopify apps use to tell merchants what happened and what to do next: banners, inline messages, toasts, modals, and page-level empty/error/loading states. For component prop reference and codegen, see reference/component-patterns.md. For copy mechanics (voice, plain language, reading level), see reference/content-and-voice.md. For color tokens and contrast numbers, see reference/layout-and-visual-design.md.

## The two alert categories

Every alert originates from one of two places, and knowing which one you're building for tells you most of what you need (shopify.dev/docs/apps/design/user-experience/alerts):

- **System alerts**: initiated by the app or platform, independent of anything the merchant just did. Examples: lost network connection, a planned upgrade, a subscription expiring soon.
- **Task alerts**: initiated in direct response to something the merchant just did. Examples: a form submitted successfully, a problem uploading a file, information that's incorrect or doesn't match the expected format.

Get this categorization right before picking a pattern. A common failure mode is treating a task alert (merchant just clicked Save) like a system alert (a banner that sits there until dismissed) — it reads as the app not trusting its own confirmation.

## Selection matrix

| Situation | Pattern | Why |
|---|---|---|
| System-level state the merchant should know about, not blocking work (degraded connection, planned upgrade, expiring subscription, informational note) | **Banner** | Page-level, persists until dismissed, can carry an action button or link (shopify.dev/docs/apps/design/user-experience/alerts) |
| Merchant just completed an action and needs quick, transient confirmation with no further step required | **Toast** | Short-lived, slides in/out, disappears automatically or is dismissed; success-only per Shopify's own guidance (shopify.dev/docs/apps/design/user-experience/alerts) |
| Merchant action failed and the failure blocks forward progress on a specific field | **Inline field error** | Placed as close to the source as possible, persists until the merchant resolves it (shopify.dev/docs/apps/design/user-experience/alerts) |
| An exception on one row of a list, or a scoped issue inside a card/section | **Inline alert scoped to that container** (badge web component for list rows) | Draws attention to the specific exception, encourages clicking through, leads with what went wrong |
| Success feedback that is delayed, persists across a session, or needs a CTA | **Banner (success, green)** | Toasts are for immediate merchant-initiated confirmation only; anything with a next step belongs in a banner (shopify.dev/docs/apps/design/user-experience/alerts) |
| A small, contained interaction (confirm a destructive action, pick from a short list, edit one field) | **Modal** | Modals are for small/contained interactions, not for surfacing error messages themselves (see below) |
| Error display in general | **Never a modal** | Only place an error message inside a modal if the modal itself is the thing failing (shopify.dev/docs/apps/design/user-experience/forms, shopify.dev/docs/apps/design/user-experience/alerts) |
| A resource list, page, or feature has genuinely nothing yet (first run, nothing created) | **Page-level empty state** | Clear explanation of what will appear plus a prominent CTA — this is an onboarding moment, not a failure (shopify.dev/docs/api/app-home) |
| A fetch, load, or background action failed and the merchant has no usable data as a result | **Page-level error state**: banner (or section-level equivalent) with an inline Retry action | Never silently fall through to a generic empty state — a failed fetch and "nothing here" are different facts and need different UI |
| A list/search/filter combination returns zero rows, but data does exist | **Filtered-to-zero state**, not an empty state | Needs a "clear filters" affordance, not onboarding copy — treating this as an empty state hides the fact that filters are the reason nothing is showing |

Notes on the matrix:
- Banner vs inline is a persistence-and-scope question: banner is page/card/modal-scoped and dismissible; inline lives directly on the offending element and persists until resolved.
- Toast vs banner for success is a duration-and-agency question: did the merchant just do this (toast), or is this state going to sit around / need a next action (banner)?
- Never use a modal as your error-display mechanism. If a modal workflow itself throws an error, show that error inside the modal; don't use a modal to announce an error that happened elsewhere on the page.

## Severity and tone hierarchy

Shopify's admin color roles (green = success, yellow = caution, orange = warning, red = critical) are semantically fixed, not decorative choices. The full meaning-per-color and never-use-for guidance lives in reference/component-patterns.md ("Badge and Banner tone semantics") — treat that as the canonical table and don't re-derive it here.

Rules that apply across every severity level when building alerts specifically:

- Never rely on color alone to convey an alert's meaning. Pair every warning and error with an icon and with text, not color alone (shopify.dev/docs/apps/design/visual-design, shopify.dev/docs/apps/design/user-experience/alerts). This is also a WCAG 2.1 AA expectation and an explicit Built for Shopify design requirement.
- Maintain at least 4.5:1 background-to-text contrast (WCAG AA) for all alert text, including inside banners with tinted backgrounds — tinted banner backgrounds are exactly where contrast mistakes hide. See reference/layout-and-visual-design.md for the full contrast rules.
- `s-badge`'s `tone` plausibly includes `neutral | info | success | caution | warning | critical` based on adjacent corpus evidence, but this isn't exhaustively confirmed — `caution` in particular only appears in the research corpus as a color-role name, not as an attested literal tone string. Verify the exact enum with `validate_component_codeblocks` before shipping. What is confirmed: there is no `"attention"` value — don't invent one. The badge component is the one named in Shopify's own guidance for inline warnings and inline errors inside list rows.
- Banner tone values seen in the research corpus include `info`, plus the prose categories success/warning/critical described above; treat the full enum as needing verification with the validate-before-ship handoff (see "Implementation notes" below) rather than assuming a 1:1 match with the badge enum.
- Warning banners are yellow and should be used intentionally — seeing one is stressful for merchants, so don't reach for a warning tone by default when info would do.
- Critical banners are red and reserved for problems that must be resolved immediately before the merchant can complete a task; use them sparingly and pair them with troubleshooting steps and a clear path to support.

## Dismissal and persistence patterns

- **Dismissible by default.** Make a banner dismissible unless it contains critical information the merchant needs to resolve before moving forward (shopify.dev/docs/apps/design/user-experience/alerts).
- **Dismissal must stick.** A dismissed banner should not reappear within the same session (shopify.dev/docs/apps/design/user-experience/alerts), and for homepage-level dismissible sections (banners, setup guide, callout cards, featured-apps) persist the dismissed/expanded state via local storage or your database so it survives a reload or a return visit (shopify.dev/docs/api/app-home/patterns/templates/homepage). A banner that resurrects itself on every navigation reads as broken, not persistent.
- **Toasts don't need persistence.** They are transient by design; dismiss-or-auto-expire is the whole point. Use the App Bridge Toast API (`shopify.toast.show()`) rather than hand-rolling a toast component — verify exact call signature via the validate-before-ship handoff before shipping.
- **Inline errors persist until resolved**, not until a timeout. Never let an error message auto-disappear after a timer (this is an explicit Built for Shopify rejection reason) — the merchant needs the message to still be there when they come back to fix the field.
- **Setup guides, callout cards, and onboarding checklists** persist both their expanded/collapsed state and their completion progress across sessions — treat this the same as banner dismissal persistence. See reference/onboarding.md for the full setup-guide pattern.

## One banner max: standing information belongs in a section

This is a load-bearing discipline, not a style preference:

- **Only one banner should be visible at a time**, anywhere on a page (shopify.dev/docs/api/app-home/patterns/templates/homepage). Built for Shopify explicitly flags two or more banners in close proximity — stacked at the top of a page or within one card — as a design-quality failure.
- If you find yourself wanting a second banner, ask whether the second one is actually **standing information** (a permanent fact about the feature, a persistent explainer, a "how this works" note) rather than a **live alert**. Standing information belongs in a plain section or paragraph, not a banner. Banner blindness sets in fast: merchants learn to skip past anything shaped like a banner within their first few sessions, so reserving banners strictly for time-bound, dismiss-once alerts is what keeps them noticeable when it actually matters.
- A useful test before adding any banner: "if a merchant dismisses this and never sees it again, have they lost anything they needed?" If the answer is yes, it's standing information and belongs in a section (with a heading, not a colored strip) rather than a banner they'll train themselves to ignore.
- This same discipline is why informational banners specifically should be scoped to lower-priority content and always dismissible (shopify.dev/docs/apps/design/user-experience/alerts) — they are the tone most tempting to overuse for content that should just be prose in a section.

## Interaction-state completeness for alerts and feedback

Every surface that can show an alert needs all four states designed, not just the happy path. This is the alert-specific version of the general loading/empty/error/success completeness rule.

### Loading

- Loading skeletons should match the shape of the loaded layout, not a generic spinner dropped in the middle of the page — a skeleton that mimics the eventual card/table/metric layout keeps the merchant oriented while data resolves.
- Where a spinner is genuinely appropriate (a small, localized wait inside a button or inline control), use `s-spinner` with a `size` value and a mandatory `accessibilityLabel`. Only `size="base"` is confirmed in the research corpus — treat `size="small"` as unconfirmed rather than confirmed-invalid, and verify the exact enum with `validate_component_codeblocks` before shipping.
- Don't stack loading indicators: a skeleton section that also contains a spinning `s-spinner` inside it is a state-design bug, not two states cooperating.
- A loading state should never look identical to an empty state — merchants shouldn't have to guess whether nothing has loaded yet or whether nothing exists.

### Empty

- Distinguish a **genuine empty state** (nothing has been created yet) from a **filtered-to-zero state** (data exists, current filters/search hide it). They are different facts and need different UI (see the selection matrix above).
- A genuine empty state needs a clear explanation of what will appear here once it exists, plus one prominent call to action — this is the Empty state composition (shopify.dev/docs/api/app-home). Use cases: first-item onboarding, feature activation prompts, a resource that hasn't been used yet.
- A filtered-to-zero state needs a "clear filters" (or equivalent) affordance instead of onboarding copy. Showing "Get started by creating your first X" when the merchant has simply filtered too aggressively is confusing and wastes their time re-reading onboarding copy they've already dismissed once.

### Error

- Every fetch or action failure must surface a banner (or scoped inline equivalent) with an inline Retry action. Never let a failed request silently fall through to a generic empty state — that hides the fact that something went wrong and makes the merchant think there's simply no data.
- Error messages should be red, tell merchants what happened, and offer a path forward rather than being scary or confusing (shopify.dev/docs/apps/design/user-experience/alerts, Built for Shopify design requirements). Avoid technical jargon and scary language.
- Avoid humor, idioms, or other phrasing that might not translate correctly — this compounds with the general rule that copy must survive translation into longer languages (German runs roughly 30% longer than English).
- Placement follows scope: a field-level validation error goes directly below the affected field; a card/section/modal-scoped error goes near the top of that element; an error that's broader than one nested component goes at the top of the broader affected element rather than being buried inside a deeply nested child.
- Timing for form validation: wait until keyboard focus leaves the field before showing a validation error. Showing an error while the merchant is still typing causes confusion and reads as the app second-guessing every keystroke.
- Never let an error message auto-dismiss after a timeout — this is an explicit Built for Shopify rejection reason, and it defeats the purpose of an error the merchant hasn't acted on yet.
- Don't nest error messages too deeply in the app's hierarchy, and don't use modals to display errors (only place an error inside a modal if the modal itself is the thing that failed).

### Success

- Success feedback for something the merchant just did themselves belongs in a toast, not a banner. Toast messages should be three words or fewer.
- Success feedback that is delayed, persistent across a reload, or carries a call to action belongs in a green success banner with the next step spelled out. Don't use a success banner when there's no CTA and no delay — that's a toast's job.
- Success states should include enough context that the merchant doesn't have to guess what changed (e.g. what specifically was saved, published, or synced), while staying within the three-word toast budget or a short banner headline.

## Alert copy discipline

Full copy rules live in reference/content-and-voice.md; the alert-specific subset:

- Lead with what went wrong, not with an apology or a generic "Something went wrong" when a more specific cause is known.
- Use a single, consistent noun/verb for the same concept across every alert in the app — don't call the same failure a "sync issue" in one banner and a "connection problem" in another.
- Keep error and warning copy scannable: short sentences, no large blocks of text, no idioms that don't translate cleanly.
- Design copy at English length but verify it still fits, and still reads well, once translated to a longer language — a banner or toast that only works at English character counts will clip or wrap awkwardly elsewhere.

## Implementation notes and handoff

- Confirmed component/API touchpoints from the research corpus: `s-banner` (`heading`, `tone`, `dismissible` attributes), `s-badge` (`tone` attribute, used for inline list-row warnings/errors — exact enum unconfirmed beyond what's discussed above), `s-spinner` (`size`, `accessibilityLabel` attributes — only `size="base"` confirmed), and the App Bridge Toast API (`shopify.toast.show()`). Treat any prop or enum value not explicitly listed here as unverified — validate every `s-*` markup block with `validate_component_codeblocks` before shipping, per the shopify-dev MCP handoff.
- If you build any custom overlay adjacent to these components (a bespoke toast stack, a custom inline popover for an error), make sure it deliberately layers above whatever it needs to sit in front of — this matters most for stacked toasts or an error tooltip that needs to sit above a modal. There's no Shopify-documented z-index scale to defer to here, so treat this as general frontend practice you're importing, not Shopify guidance.
- All interactive dismiss controls (banner close button, toast dismiss) need visible keyboard focus and must be operable via keyboard, not just click/tap, per the accessibility non-negotiables.
- Respect `prefers-reduced-motion` on any custom-authored slide/fade transition for toasts or banners — Shopify's own toast "slides in and out," but a merchant with reduced-motion preferences should get an instant or minimally-animated equivalent.
- For design judgment on where a given page's *one* dominant primary action lives relative to any alert on that page, see reference/navigation.md — an alert's action button (e.g. a banner's CTA) should never compete with the page's own primary action for the same intent.
