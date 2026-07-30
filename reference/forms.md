# Forms

Forms are where merchants create and edit things. In most Shopify apps they are the single most-used surface in the product, more used than any dashboard. Treat form quality as a first-class design problem, not a task you knock out after the "real" UI is done.

## Design read (do this before laying out any form)

One line, before you write markup: what entity does this form create or edit, is it a full page or a scoped modal interaction, and what's the one thing that happens when the merchant hits save. If you can't state that in one sentence, the form is trying to do too much — split it.

## Layout

**One form, one entity, one dedicated page.** Mirror the Shopify admin's own convention: a single page dedicated to editing a single entity (one page for a product, one page for a variant), not a multi-entity form crammed onto one screen (shopify.dev/docs/apps/design/user-experience/forms). This is also why large forms don't belong in modals — modals give merchants a small, capped workspace; a form with real depth needs a full page. Reserve modals for genuinely small, contained interactions (confirm, rename, pick one option), never for a form with more than a handful of fields.

**Grouping threshold: more than five inputs, add structure.** Past five inputs, don't let the form run on as one flat stack. Either:
- Break it into titled sections inside a single card, or
- Split it into multiple cards, each with its own header.

Pick sections-in-one-card when the fields are all facets of the same conceptual object (e.g. one product's title, description, and media). Pick multiple cards when the groups are more independent and a merchant might reasonably fill in one and skip another for now (e.g. "Basic details" vs "Search engine listing" vs "Shipping").

**Spacing comes from the stack primitive, not manual margins.** Use `s-stack` (`direction="block"`, consistent `gap`) to lay out fields vertically with even rhythm — don't hand-space fields with ad hoc padding. See reference/layout-and-visual-design.md for the full spacing/density system and when to go looser vs tighter.

**Settings-style forms get the settings layout.** For configuration/preferences forms specifically (as opposed to "create a new X" forms), use the settings pattern: a thinner left column carrying the section title + one line of description, a wider right column carrying the actual fields. This gives merchants context for *why* a setting exists right next to the control, instead of a bare label. See reference/layout-and-visual-design.md for the full settings-layout treatment.

**Progressive disclosure for conditional fields.** When a field's presence or meaning depends on an earlier answer, don't show it up front — reveal it only once the earlier field makes it relevant. This is the difference between a long form that overwhelms on load and one that reveals itself as the merchant proceeds (shopify.dev/docs/apps/design/user-experience/forms).

**Label placement is not a per-form decision.** Polaris form components (`s-text-field`, `s-select`, `s-checkbox`, etc.) take `label` as a required-in-practice prop and render it in a fixed position as part of the component itself — you don't get to float it, hide it decoratively, or move it beside the field. Don't fight this. The one place you do make a placement call is helper and error text (below).

**Helper text lives under the field, in the field's own slot.** Use the component's own helper-text affordance rather than inventing a freestanding paragraph next to the field:

```
<s-text-field
  label="Title"
  name="title"
  value={title}
  details="Minimum 10 characters recommended"
></s-text-field>
```

`details` (or the equivalent helper-text slot on whichever field component you're using) keeps helper copy visually and programmatically tied to its field. Verify the exact prop name against current docs before shipping (shopify-dev MCP `validate_component_codeblocks`) — treat `details` as representative, not gospel.

**Error text lives directly below the field it belongs to, every time.** Never float a field's error message elsewhere on the page, never collapse multiple fields' errors into one summary banner at the top instead of per-field text (a summary can supplement per-field errors, never replace them), and never nest an error message deeper than the field's own container. If an error is genuinely page-level (e.g. a save failed for a reason unrelated to any single field), that's a banner, and it belongs in the alert vocabulary covered in reference/alerts.md, not force-fit into a field.

## Validation timing

**No continuous validation. No auto-save.** This is stated flatly in the Shopify design guidelines: continuous data validation or auto-save on forms is incongruous with the standard Shopify admin save UX (shopify.dev/docs/apps/design/user-experience/forms). Merchants expect an explicit, reviewable save step, not a form that's silently rewriting itself or scolding them mid-keystroke.

**Never validate while the merchant is still typing.** Wait until focus leaves the field (blur) before showing a validation error. Showing an error on every keystroke reads as broken and causes real confusion — a field that's "wrong" for the first four characters of a valid email address is not an error yet.

**On submit, move focus to the problem.** If a submit attempt fails validation, place focus on the error feedback (the first invalid field, or an error summary if you have one) rather than leaving focus wherever it was and hoping the merchant notices a change somewhere on the page.

**Errors persist until the merchant actually fixes them.** Don't auto-dismiss a field error after a timeout, and don't use a toast to report a form validation error — toasts are for short, transient confirmations, not for something the merchant needs to act on. An error that can vanish before it's read is a bug, not a UX nicety.

**Success feedback depends on whether it's instant or not.** If save succeeds immediately with nothing further for the merchant to do, use a toast. If success is delayed, persistent, or comes with a next-step CTA, use a banner instead. Full pattern detail (banner vs inline vs toast, tone, dismissal rules) lives in reference/alerts.md — this file only covers the form-specific slice of that decision.

**Save through the Contextual Save Bar, not a bespoke submit button embedded in the page body.** Wire the form to the Save Bar (`data-save-bar` on the `<form>` element, or the equivalent Save Bar API for your setup) so unsaved-changes UX matches the rest of the admin — merchants get the same "you have unsaved changes, save or discard" affordance everywhere, including inside your app:

```
<form data-save-bar onSubmit={handleSave} onReset={handleDiscard}>
  <s-section heading="Basic details">
    <s-stack direction="block" gap="base">
      <s-text-field label="Title" name="title" value={title} required></s-text-field>
      <s-text-area label="Description" name="description" value={description} rows={4}></s-text-area>
    </s-stack>
  </s-section>
</form>
```

This matters even for forms that live inside a modal — the Save Bar convention still applies rather than a modal-local "Save" button pretending to be the whole admin's save model.

## Shopify component selection guide

Pick the field component by what the data actually is, not by habit. Verify exact props/enums with `validate_component_codeblocks` before shipping — this table is guidance on which component to reach for, not a locked prop reference (that's owned by the shopify-polaris-app-home / shopify-polaris-admin-extensions marketplace skills).

| Data being collected | Reach for | Notes |
|---|---|---|
| Short free text (title, name, SKU) | `s-text-field` | Default text input. |
| Long free text (description, notes) | `s-text-area` | Use `rows` to size for expected content length, not to force scrolling. |
| Email address | `s-email-field` | Set `autocomplete="email"`. |
| Web address | `s-url-field` | Set `autocomplete="url"`. |
| Password / secret | `s-password-field` | Set `autocomplete` to `current-password` or `new-password` as appropriate; use `minLength` if you enforce one. |
| Integer / quantity | `s-number-field` | Set `inputMode="numeric"` and `min`/`max`/`step` for the real constraints, not just for show. |
| Currency amount | `s-money-field` | Don't reach for a plain text field with a "$" prefix hack. |
| One choice, options can stay hidden until opened | `s-select` | Good when space is tight or the option list is long. |
| One (or multi) choice, options should stay visible for scanning/comparison | `s-choice-list` / `s-choice` | Better when the list is short and merchants benefit from seeing all options at once. |
| Boolean that takes effect immediately (a feature flips on right now) | `s-switch` | Communicates "this acts now," not "this saves later." |
| Boolean that's part of a batch the merchant will explicitly save | `s-checkbox` | Communicates "this is one input among several, reviewed together." |
| Date, typed entry | `s-date-field` | Pair with `s-date-picker` when a calendar picker adds real value over typing. |
| Color | `s-color-field` / `s-color-picker` | Use `alpha` if transparency is a real option, not by default. |
| File upload | `s-drop-zone` | Set `accept` to the real constraint; don't leave it wide open if the backend only handles specific types. |
| Live search-as-you-type, distinct from a field whose value gets submitted | `s-search-field` | Not a substitute for a committed form field. |

Don't invent exact enum values (e.g. every valid `tone` on a banner referenced from a form, or every `autocomplete` token) from memory — when the research corpus doesn't nail down the value you need, phrase it as "verify against current docs" and use the validate-before-ship handoff instead of asserting it as fact.

## Accessibility rules specific to forms (non-negotiable, not aesthetic choices)

- **Never use a placeholder as a label.** A placeholder disappears the moment the merchant starts typing, so a field with only placeholder text has no label at all once it's in use — this is a real, common accessibility failure. Every field gets a real `label` (via the component's `label` prop, `aria-label`, a visually-hidden label element, or a visible label with a `for` attribute pairing) (shopify.dev/docs/apps/build/accessibility). Placeholder text, if you use it at all, is only for a short example-format hint ("e.g. 4111 1111 1111 1111"), never the field's only name.
- **Required fields use the `required` attribute**, not just a visual asterisk that a screen reader has no way to parse.
- **Set `autocomplete` on fields where a real browser autocomplete category applies** (email, url, current-password, etc.) — it's a small thing that meaningfully speeds up merchants filling in known values.
- **Error text must be real DOM text associated with its field, every time an error exists.** A red border or an error icon alone is not an error message — pair it with visible text, and reference that text from the input via `aria-describedby` so assistive tech announces it together with the field. An input that's visually flagged red with no accompanying text is a Built for Shopify rejection reason, not just a nice-to-have gap.
- **On submit failure, place focus on the feedback message** so keyboard and screen-reader users land where the problem is instead of having to hunt for it.
- **Visible focus on every field, always** — never suppress the focus ring for aesthetic reasons. This applies to every input, select, checkbox, switch, and button in the form, not just the primary action.
- **Announce success/error state changes via `aria-live`** so a screen-reader user gets the same "it worked" / "it didn't" signal a sighted merchant gets from color and iconography.
- **Write labels and error copy that survive translation growth.** German commonly runs ~30% longer than English. A label or inline error that's tightly hand-fit to English length will clip, wrap badly, or get truncated once translated — don't design against a fixed-width assumption.
- **Give every field an unambiguous label, including units.** A field labeled just "Time" with no unit is a real, documented rejection pattern — merchants (and translators) shouldn't have to guess.

## Anti-slop specifics for forms

- Don't add a cosmetic "looks validated" green checkmark before real validation has actually run against real data — a checkmark that appears just because a field is non-empty is decoration, not feedback.
- A form section should have at most one primary-styled action. Don't style both "Save" and some secondary action (e.g. "Preview," "Duplicate") as equally prominent primary buttons — that's the same "two CTAs with the same intent" problem called out for pages generally, applied at the form level.
- Don't build a "setup wizard" feel into an ordinary edit form by adding fake step indicators or progress bars where there's no actual multi-step sequence — save that pattern for real onboarding flows (see reference/onboarding.md).
- Don't scatter unrelated settings into one form just because they technically save together on the backend — if merchants conceptually think of them as separate concerns, give them separate cards/sections even if one submit handles both.

## Handoff

Once the layout and component choices above are decided, hand implementation to the shopify-polaris-app-home or shopify-polaris-admin-extensions marketplace skill (whichever surface applies) for exact prop names, enum values, and codegen — those skills run the mandatory search-then-validate loop this file intentionally does not duplicate. Before shipping any `s-*` markup for a form, run it through `validate_component_codeblocks`. For the full alert/banner/toast vocabulary referenced above for success and page-level errors, see reference/alerts.md. For overall page structure this form sits inside (single-column vs settings vs two-column), see reference/layout-and-visual-design.md.
