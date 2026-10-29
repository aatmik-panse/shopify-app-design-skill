# Onboarding

Onboarding is the process a merchant follows to set up your app for the first time. It runs entirely inside Register A (the embedded admin surface, see SKILL.md). There is no marketing register here, no "wow" welcome takeover, no brand hero. The job is narrower and more important: get the merchant from install to their first real, working action as fast as honestly possible (shopify.dev/docs/apps/design/user-experience/onboarding).

Success is not "the merchant saw a nice welcome screen." Success is: the merchant knows what to do next, and the app's state on disk (not just on screen) reflects that they did it (shopify.dev/docs/apps/design/user-experience/onboarding).

## Start with a design read

Before wireframing a first-run flow, write one line:

> "First-run home for [app]. Merchant state: just installed, zero data, N required setup steps. Primary next action: [connect X / create first Y]."

If you can't fill in "merchant state" with something concrete and checkable (a scope granted, a webhook received, a row in your own database), you are not ready to design the onboarding flow yet. Onboarding design is state design first, screen design second.

## Welcome, don't market

The purpose of onboarding is to welcome the merchant, make them eager to use the app, make them comfortable, and set expectations immediately (shopify.dev/docs/apps/design/user-experience/onboarding). That is a Register A job, not a Register B job:

| Do (Register A welcome) | Don't (Register B leaking in) |
|---|---|
| One short heading stating what the app does for this store, plus one primary action | A hero banner with marketing copy, illustration, and a "Learn more" link competing with the real CTA |
| Set expectations: "Here's what we'll set up, in this order" | Oversell language, guaranteed outcomes, or urgency framing (see reference/creative-playbook.md and the Built for Shopify anti-dark-pattern rules) |
| Fits inside the ordinary page layout, uses the app's own visual system | Custom display fonts, saturated brand-color washes, countdown timers |

This welcome moment is also strictly a first-run thing. The instant the merchant has completed setup (or dismissed it), the home page reverts to being a normal, working dashboard. A returning user should never see first-run welcome copy again. See reference/app-home.md for what fills that space on a returning visit.

## Hard constraints (apply to any onboarding flow)

These come directly from Shopify's design guidance and the Built for Shopify design requirements. Treat them as non-negotiable, not stylistic preference:

- **Five steps, maximum.** More steps measurably increases drop-off (shopify.dev/docs/apps/design/user-experience/onboarding). If your setup genuinely needs more than five actions, group them or cut the ones that aren't required for first value.
- **Ask for nothing you don't need.** Request merchant information only if it's necessary to proceed (shopify.dev/docs/apps/design/user-experience/onboarding). Every field in a first-run form needs a reason a merchant could state back to you. "What types of products do you sell?" with no stated purpose is a rejection-worthy pattern under Built for Shopify (shopify.dev/docs/apps/launch/built-for-shopify/requirements#design).
- **Make it dismissible if it isn't essential.** Use a clear, accessible dismiss control (the conventional pattern is an "X" in the upper right, following the Polaris Cancel icon) (shopify.dev/docs/apps/design/user-experience/onboarding). If onboarding is truly mandatory (the app is unusable without it), don't fake a dismiss control that does nothing; just don't offer one.
- **Never imply installing another app is a required step.** A setup-guide primary button that installs a different app is an explicit Built for Shopify rejection reason (shopify.dev/docs/apps/launch/built-for-shopify/requirements#design).
- **Stay visible and concise.** Onboarding that's collapsed, buried, or too wordy to guide a merchant to completion fails Built for Shopify's "Helpful onboarding" requirement (shopify.dev/docs/apps/launch/built-for-shopify/requirements#design).
- **Leave a way out after completion.** Once a merchant finishes onboarding, they need a way to dismiss or collapse it permanently. Don't leave a completed checklist as a permanent fixture with no dismiss path.
- **One primary action per screen**, derived from real state, same as every other screen in the app (see SKILL.md non-negotiables).

## Progressive activation: gate every step to real state

This is the part that separates a real setup guide from decoration. The single most common failure mode in AI-generated (and plenty of human-built) onboarding is a checklist that is cosmetic: every box is checkable regardless of whether the merchant actually did anything, or the "complete" state is driven by "did they visit this page" instead of "did the thing actually happen."

| Cosmetic (wrong) | Gated to real state (right) |
|---|---|
| Step marked complete when the merchant clicks a button that just says "Got it" | Step marked complete when the underlying action actually succeeded: a scope was granted, a webhook fired, a row was written, an API call returned success |
| Step marked complete because the merchant navigated to the relevant settings tab | Step queries or listens for the real signal (e.g. app.extensions() to confirm a theme app embed or app block is actually activated in the merchant's theme, per Built for Shopify guidance at shopify.dev/docs/apps/launch/built-for-shopify/requirements#design) |
| Progress bar resets to 0 on every page load because nothing is persisted | Progress and per-step completion are persisted server-side (or at minimum local storage) and rehydrated on every load, matching what's actually true |
| Checklist stays visible and "in progress" forever because there's no path to 100% | Every step has a checkable, unambiguous completion signal; if a step can't be verified programmatically, it isn't a good onboarding step, reconsider it |
| Dismissing onboarding leaves a blank or near-blank home page | The home page is guaranteed to contain live, useful content after dismissal (Built for Shopify explicitly requires the home page to remain non-static and useful once every dismissible element has been dismissed, not just links or a static welcome message) |

Practical rule: for every step in a setup checklist, write down the exact query, event, or condition that flips it to "done" before you build the UI for it. If you can't write that condition down, the step is step-fakery and will read as fake to merchants and to Built for Shopify reviewers alike.

Two more state-gating details worth calling out explicitly:

- **Don't show the checklist before you know the real state.** On first paint, you often don't yet know whether a merchant has zero products, an unactivated theme extension, or a fully-configured store. Show a loading skeleton shaped like the eventual checklist while that state resolves (see SKILL.md non-negotiables on skeletons), not a flash of an onboarding card that then disappears once you learn setup was already done. That flash reads as buggy, and Built for Shopify explicitly penalizes UI that flickers or repeatedly loads in and out.
- **A completed step should not be re-offered as incomplete due to a stale read.** If your state check has any lag (webhook delay, eventual consistency), bias the UI toward "we're confirming" rather than falsely telling a merchant to redo work they already did.

## The setup guide composition itself

For the actual composition (the checklist card, its heading, progress text, expand/collapse behavior, and per-step layout), don't rebuild this from scratch: see reference/app-home.md, which owns the setup-guide composition pattern and its placement inside the home page layout. This file governs whether and how you gate each step to real state; reference/app-home.md governs how the card is composed and where it sits among the home page's other sections. For exact component props and any `s-*` markup, validate against reference/component-patterns.md and Shopify's `validate_component_codeblocks` tool before shipping (see SKILL.md handoff section).

What belongs here, not there: the five-step cap, the dismissibility rule, the completion-signal design for each step, and the "don't fake it" discipline above.

## When a checklist isn't the right shape

Not every first run needs a multi-step checklist. Two lighter-weight patterns from Shopify's own guidance:

- **Dismissible step cards.** For very short flows, present each step as its own small, dismissible card with a title, one line of body copy, and a single action button, rather than one big composite checklist (shopify.dev/docs/apps/design/user-experience/onboarding).
- **A choice of starting points.** When the first real decision a merchant faces is "how do I want to begin," offer a small set of clear options instead of a linear checklist, for example: edit an existing item, start from a template, or start from scratch (shopify.dev/docs/apps/design/user-experience/onboarding). Keep the choice to a handful of clearly labeled options; this is a decision screen, not a form.

For setup that realistically takes longer than a merchant has time for right now, give them a "Remind me later" secondary action next to the primary "Continue setup" action, so you don't force an interruption to their actual workflow (shopify.dev/docs/apps/design/user-experience/onboarding). Persist that snooze the same way you persist step completion: a stale "remind me later" that reappears every page load, or one that never reappears at all, are both bugs.

## The empty state as the onboarding moment

The first time a merchant lands on a list, table, or resource page with nothing in it yet, that blank moment is itself an onboarding opportunity, not a dead end. Use `s-empty-state` (shopify.dev/docs/api/app-home/latest/web-components/feedback-and-status-indicators/empty-state). Explain what will appear here, and put one prominent action in `primary-action`.

```
<s-empty-state heading="No campaigns yet">
  <s-text slot="subheading">Campaigns you create will appear here, with send status and open rate.</s-text>
  <s-button slot="primary-action" variant="primary">Create campaign</s-button>
</s-empty-state>
```

Get the distinction right, because this is where generic AI-dashboard output usually goes wrong: **a true empty state is not the same thing as a filtered-to-zero state.**

| State | What happened | What the UI should show |
|---|---|---|
| True empty state | Nothing exists yet for this merchant (first run, no data created) | Onboarding-style explanation + one prominent CTA to create the first item |
| Filtered-to-zero state | Data exists, but the current filter/search/sort combination matched nothing | The same `s-empty-state`, with a plain "No results" line and a Clear filters action, never onboarding copy |

Reusing first-run onboarding copy ("Get started by creating your first campaign") when a merchant has simply filtered their existing 40 campaigns down to zero is confusing and condescending. Check which case you're actually in (is the underlying resource count zero, or is the filtered/searched count zero) before choosing which copy and which action to show (see SKILL.md non-negotiables).

The first-run empty state is also a legitimate place to spend your one signature creative moment for the page (see SKILL.md "spend your boldness in one place"): a bespoke illustration in the `graphic` slot, or a small entrance animation, is earned craft, gated behind `prefers-reduced-motion` like any other motion in Register A. Don't spend that same budget again on the setup guide card two inches above it; pick one moment per page.

## Copy and accessibility specifics for onboarding

- Write onboarding copy at the reading level and brevity called for on any Register A surface (see reference/content-and-voice.md), but hold it to an even tighter bar here: onboarding is the first impression, and a merchant who doesn't understand step 1 won't reach step 5.
- CTA labels follow verb+noun ("Connect your store", "Create first product"), never a bare "Next" or "Continue" with no object.
- German and other longer-language translations run roughly 30% longer than English; a step title that just fits in English will wrap or truncate in German. Leave room, don't hard-wrap the layout to English string lengths.
- On first reference to your app within a welcome message, use its proper name; "we" is fine for the rest of that same message (shopify.dev/docs/apps/design/content).
- If a dismiss control is icon-only, it needs an accessible label (for example "Dismiss setup guide"), not just a bare icon.
- If any part of onboarding opens in a modal or drawer, follow the standard modal rules: focus moves to the element labeling it on open, keyboard navigation stays trapped inside while open, Esc closes it and returns focus to whatever launched it (see SKILL.md non-negotiables and shopify.dev/docs/apps/build/accessibility).
- Status changes during onboarding (a step flipping to complete, a progress count updating) should be announced to screen reader users the same way any other dynamic status change is, consistent with the app's general `aria-live` handling for notifications (shopify.dev/docs/apps/build/accessibility); verify the exact mechanism against current docs before shipping.

## Before you ship: onboarding-specific checklist

- [ ] Five steps or fewer; if more were needed, some got cut or grouped.
- [ ] Every requested field has a stated reason.
- [ ] Every checklist step has a written-down, real completion signal, not "merchant clicked a button that says done."
- [ ] Checklist state is persisted and rehydrates correctly on reload (no false resets, no false stale completions).
- [ ] Onboarding is dismissible (if non-essential) with an accessible dismiss control, and stays dismissed.
- [ ] No step implies installing a different app is required to finish setup.
- [ ] Home page after full dismissal still shows real, live, useful content, not a blank shell or a static "Welcome!" message.
- [ ] The welcome/marketing tone present on day one is gone on every subsequent visit.
- [ ] Any list/table's zero-state has been checked against both "truly empty" and "filtered to zero," with different copy and different CTA for each.
- [ ] Loading skeleton, not a flash of onboarding UI, covers the moment before real state is known.
- [ ] Copy self-audit done: reread every onboarding string out loud, check for AI-ish cuteness, unclear referents, or copy that only fits at English length.
