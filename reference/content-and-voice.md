# Content and voice

This file covers microcopy: what an app's words should do, on Register A (the embedded admin surface). For component selection and props see reference/component-patterns.md. For layout and information architecture see the relevant layout/navigation reference file. For Register B (App Store listing, marketing site) copy rules, see the marketing-specific reference file, not this one.

Content quality is a design requirement, not a polish pass. Shopify states it directly: "Writing and designing content in a thoughtful way makes your app usable, accessible, and readable" (shopify.dev/docs/apps/design/content). Bad copy is a UX bug with the same severity as a broken layout.

## Voice vs tone

Voice is what stays constant no matter what: your app's personality, vocabulary, and level of formality. Tone is what flexes with context: a success toast and an error banner should sound like the same app, but not the same mood (shopify.dev/docs/apps/design/content). Don't let an app sound clinical in Settings and jokey in onboarding. Pick one voice and let tone do the situational work.

## Write from the merchant's side of the screen

The single biggest voice mistake is writing copy from the app's point of view (what the app did, what the app thinks) instead of the merchant's point of view (what the merchant needs to know, what the merchant should do next).

Shopify's own framing for this is "guide, don't prescribe": write in a way that puts the merchant at the center and in control, and give them enough information to make the right decision on their own rather than dictating the decision for them (shopify.dev/docs/apps/design/content). That has direct copy consequences:

| Prescriptive (app-centered) | Guiding (merchant-centered) |
|---|---|
| "You must connect a payment provider before continuing." | "Connect a payment provider to start accepting orders." |
| "Invalid input." | "Enter a value between 1 and 100." |
| "We recommend enabling this." | "Enabling this lets customers see stock levels on the product page." |
| "Action required." | "3 products are missing images. Add images to include them in your next campaign." |

Notice the guiding column always answers "so what happens if I do this / don't do this" in the same sentence. That's the tell for merchant-centered copy: it states the consequence, not just the instruction.

This also governs self-reference. On first mention of your app or company within a content section, use its proper name; for later mentions in that same section "we" is fine to keep things concise, for example: "Puzzlify detected an issue with your shipment. We've contacted our supplier and will send you updates as we receive them." (shopify.dev/docs/apps/design/content). Don't refer to the merchant's own store or data as if it belongs to the app ("our products" when you mean the merchant's products).

## Active voice and one vocabulary per action

Shopify's content guidelines are explicit here: for calls to action, start with a strong verb that describes the action, and use active voice to clarify the subject and the action (shopify.dev/docs/apps/design/content). Passive constructions ("Your changes have been saved by the system") hide who did what and read as evasive when something goes wrong.

More important than any single sentence being active voice is vocabulary discipline across an entire flow. Shopify's rule: identify and eliminate synonyms for key concepts and actions, and use a single noun, verb, or phrase to describe a specific thing, action, or concept throughout the app (shopify.dev/docs/apps/design/content). Their own illustrative example: use the same wording ("Upload image") in both a card's descriptive text and its button, rather than mixing "Upload images" in the text with "Add photos" on the button.

This is the rule that most commonly breaks across a flow, not within a single screen. A button's action name has to match the resulting toast, banner, and any downstream status text word for word (or as close as grammar allows). If the button says "Publish," the toast says "Published," never "Live" or "Saved" or "Done." If the button says "Sync inventory," the success state says "Inventory synced," not "Update complete."

Concrete failure pattern to check for on every flow:

```
Button label:        "Generate report"
Loading state:       "Building your report…"      <- inconsistent verb ("Building" vs "Generate")
Success toast:       "Export complete"            <- inconsistent noun ("Export" vs "report")
```

Fixed:

```
Button label:        "Generate report"
Loading state:       "Generating report…"
Success toast:       "Report generated"
```

Build a small verb/noun glossary for each core action in your app (one row per action: the noun, the verb, the past-tense/result form) before writing any copy, and check every button, menu item, toast, banner, and empty-state CTA against it. This is cheap to do up front and expensive to fix once copy is scattered across a codebase.

## Empty state copy

An empty state and a filtered-to-zero state are not the same thing and must not share copy (see the non-negotiables list in SKILL.md). An empty state onboards; a filtered-to-zero state helps someone escape a filter.

| State | What happened | What the copy should do |
|---|---|---|
| True empty state | Merchant has never created a resource yet | Explain what will appear here and give a clear, prominent call to action to create the first one |
| Filtered-to-zero state | Resources exist, but the current search/filter/sort matches none of them | Say plainly that no results match the current filters, and offer a "Clear filters" or "Clear search" affordance, not onboarding copy |

Writing "Create your first campaign to get started" when the real problem is an over-narrow filter tells the merchant to do the wrong thing (create a duplicate) instead of the right one (clear the filter). Always check which state you're actually in before writing the copy, not just before choosing the component.

For true empty states, the copy has one job: explain what will appear here once the action is taken, and lead the CTA with a strong verb (shopify.dev/docs/apps/design/content combined with the Empty state composition guidance referenced in component-patterns.md). Don't pad it with brand voice or jokes; get the merchant to the action.

## Error copy: never vague, never hollow-apologetic

Shopify's alerts guidance (shopify.dev/docs/apps/design/user-experience/alerts) is unusually concrete about what error copy has to do:

- Always tell merchants what happened and offer a path forward. An error banner that states a problem with no next step is incomplete by definition.
- Lead with what went wrong, not with the app's internal framing of the problem.
- Guide merchants to a solution clearly and quickly, don't be scary or confusing.
- Avoid scary language, technical terms, and jargon.
- Avoid humor, idioms, or phrases that might not translate correctly.
- Never rely on color alone to convey severity; pair red/warning color with text and, for inline list warnings, an icon.
- Place the error as close to the problem as possible: inline, directly below the affected field, not floated to the top of an unrelated container.

The direct implication for copy is a two-part shape, not a single vague clause:

```
[what happened, in plain terms] + [what to do about it]
```

"Something went wrong" fails this test entirely: it names nothing and offers no path forward. So does "An error occurred while processing your request" (jargon, no path forward, and needlessly technical for a problem that likely has a plain-language cause). Compare:

| Avoid | Use instead |
|---|---|
| "Something went wrong." | "We couldn't save your changes because the connection was interrupted. Try again." |
| "Error: 422 Unprocessable Entity" | "That discount code is already in use. Choose a different code." |
| "Invalid input." | "Enter a number between 1 and 100." |
| "Oops! We're so sorry, something broke on our end!" | "We couldn't load your orders. Retry, or contact support if this keeps happening." |

The last row is the falsely-apologetic trap: piling on "Oops," exclamation points, and self-flagellation doesn't give the merchant anything they can act on, and it reads as performative rather than helpful. One calm, plain sentence naming the problem, followed by the fix or the retry action, beats an apology every time. This directly reflects the non-negotiable that every fetch failure needs a banner with an inline Retry, not just a sorry-shaped dead end.

Error copy checklist per message:

- [ ] Does it say specifically what failed (not just "an error")?
- [ ] Does it give the merchant something to do next (retry, fix a field, contact support)?
- [ ] Is it free of jargon, stack traces, and internal status codes?
- [ ] Is it placed next to the thing it's about, not just visually red?
- [ ] Does it avoid over-apologizing in place of being useful?

## Success copy: match the pattern to the persistence

Shopify draws a hard line between toast and banner for success feedback, and the copy register for each should match:

- Toast: for a merchant-initiated action's immediate confirmation. Keep it to three words or fewer (shopify.dev/docs/apps/design/user-experience/alerts). "Product saved." "Article published." "Settings updated." No qualifiers, no exclamation points, no restating the whole action.
- Banner: reserved for success feedback that's delayed, persistent, or carries a next step / CTA. Here the copy can (and should) do more work: state what happened and what to check or do next. "Your bulk import finished. 42 products were updated, 3 were skipped because of missing SKUs. Review skipped products."

Using banner-length copy in a toast is a common tell of unpolished copy: it forces merchants to read a paragraph in a UI element that's designed to be glanced at and dismissed. Using toast-length copy ("Done.") for a banner wastes the one component built to carry a real explanation and a CTA.

Match the toast's noun and verb to the button that triggered it, per the action-vocabulary rule above. This is the most common place a flow's vocabulary quietly drifts, because the toast is often written by someone other than whoever wrote the button.

## i18n-safety: copy that survives translation

German runs roughly 30% longer than English once translated, and many merchant-facing markets are non-English-first. Two separate risks follow from this, and both are addressed directly in Shopify's own content guidance:

**Risk 1: meaning that doesn't survive translation.** Merchants worldwide have varying literacy and many don't speak English as a first language, so content "can easily be mistranslated or misinterpreted" (shopify.dev/docs/apps/design/content). Concretely:

- Avoid jargon or overly technical language a non-native or non-technical reader won't recognize.
- Avoid idioms and phrases with indirect or ironic meanings (Shopify's own example of what to avoid: "Beggars can't be choosers"). These don't translate literally and often invert or lose their meaning.
- Avoid humor, wordplay, or "clever" phrasing in any UI copy that will be localized, per the same rule applied to alerts (shopify.dev/docs/apps/design/user-experience/alerts). A pun that lands in English is dead weight, or actively confusing, in German, Japanese, or Portuguese.
- Aim for roughly a US grade 7 reading level; check with a tool like Hemingway App (shopify.dev/docs/apps/design/content).
- Use short sentences, headings, and bullets over large blocks of text; large blocks are hard to scan in any language and compound translation risk (shopify.dev/docs/apps/design/content).

**Risk 2: layout that doesn't survive translation.** Never design or write copy that only works at English pixel width or character count:

- Don't hard-wrap text with manual line breaks tuned to English word lengths.
- Don't size a button, chip, or badge to fit an English label exactly, with no room to grow; German, Finnish, and Dutch labels routinely run longer and will clip or wrap unpredictably.
- Don't write copy that depends on an exact character count fitting a fixed-width slot (a toast capped to visually fit "Saved" will overflow when the translated string is three times longer).
- Write short, then verify the layout has visible breathing room at both ends: test with a genuinely long string (or just write a placeholder in German) before assuming a container will hold it.
- Prefer components and containers designed to reflow (wrap, truncate gracefully with an accessible full-text alternative, or grow) over ones with a fixed pixel budget.

A useful gut check when drafting any UI string: could a translator swap out every word and still produce something roughly 30% longer that fits without redesigning the component? If not, the copy or the layout (or both) need to change before shipping.

## Grammar, mechanics, and self-reference

Apps should use proper grammar throughout; for the full rules on spelling, punctuation, special characters, dates, numbers, and measurements, defer to the Polaris grammar-and-mechanics guidelines rather than improvising a house style (shopify.dev/docs/apps/design/content, polaris.shopify.com/content/grammar-and-mechanics). Within this skill's scope, the actionable takeaways are:

- Use one proper name for your app/company on first reference in a content section; "we" is fine after that, but don't oscillate between the two mid-paragraph.
- Don't duplicate content: don't add a page title under the Title bar component, don't add horizontal navigation when the App nav component is already present, and don't say the same thing two different ways on the same page (shopify.dev/docs/apps/design/content). If a card's body text and its button already say "Upload image," a nearby banner repeating "Don't forget to upload an image" is noise, not reinforcement.

## Mandatory pre-ship copy self-audit

Before any new or changed string ships, reread every visible string in the flow (not just the ones you wrote last) and check it against this list. Do this as a distinct pass, not a byproduct of writing the copy in the first place.

- [ ] **Merchant's side check.** Does every sentence read as if written for the merchant's decision, not the app's internal state? Any "we/our" framing that should be "you/your"?
- [ ] **Vocabulary consistency check.** For every action in this flow: does the button verb match the loading-state verb, the toast, and any banner or history log referring to the same action? Build or update the verb/noun glossary if this is a new action.
- [ ] **Active voice check.** Any passive constructions hiding who or what did something ("was updated," "has been removed")? Rewrite active unless there's a real reason the actor is unknown.
- [ ] **Empty vs zero-result check.** Is this actually a true empty state, or a filtered-to-zero state wearing empty-state copy? Fix the mismatch.
- [ ] **Error completeness check.** Does every error string name the actual problem and offer a next step? Cut any error that only apologizes or only names an internal code.
- [ ] **Toast/banner length check.** Are toasts three words or fewer? Is anything that needs more explanation living in a banner instead of a toast?
- [ ] **Jargon and idiom check.** Any technical terms, internal product names, or idioms a first-time or non-native-English merchant wouldn't immediately understand?
- [ ] **Humor/wordplay check.** Any pun, cute wordplay, or "clever" phrasing that risks not surviving translation or reading as unserious in an error/warning context?
- [ ] **Reading level check.** Rough US grade 7 target; run a spot check through a readability tool on any paragraph-length copy (setup guides, callouts, longer banners).
- [ ] **Length-safety check.** For every fixed-width or fixed-height container holding this copy, would a ~30% longer German equivalent still fit or reflow gracefully, rather than clip or overflow?
- [ ] **Duplication check.** Is this string saying something already said elsewhere on the same page (title bar, nearby card, nearby banner)? Cut the repeat.
- [ ] **Grammar and mechanics check.** Spelling, punctuation, capitalization, and number/date formatting consistent with the rest of the app and with Polaris grammar-and-mechanics conventions.

Treat a failed checklist item the same as a failed test: fix the copy before merging, don't ship it "for now."
