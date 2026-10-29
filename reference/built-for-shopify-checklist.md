# Built for Shopify: Design Pre-Submission Checklist

Built for Shopify (BFS) is Shopify's quality bar for the App Store. It gates a visible badge, better discovery placement, and eligibility for featured spots. A chunk of the requirements are pure design/UX judgment calls, which is why this checklist exists: walk it against the real app, in the real Shopify Admin, before submission.

Source: shopify.dev/docs/apps/launch/built-for-shopify/requirements#design (Section 4: Design). Requirement IDs below (4.1.1, 4.2.3, etc.) are quoted from that page so findings stay auditable back to the source.

**Scope of this file.** This is the pass/fail submission gate, not the design system. For the underlying "why" and the full pattern detail behind each rule, this file cross-references sibling reference files rather than repeating them:
- Exact `s-*` prop names, enums, and syntax gotchas: reference/component-patterns.md
- Full WCAG 2.1 AA item list (focus order, aria-live, form labeling, etc.): reference/accessibility.md
- Layout pattern depth (single/two-column, settings pattern, grid): reference/layout-and-visual-design.md
- Navigation depth (app nav, tabs, breadcrumbs): reference/navigation.md
- Voice, tone, reading level, copy consistency: reference/content-and-voice.md
- Alert/banner/toast/inline-error patterns: reference/alerts.md
- Form patterns (sectioning, save bar wiring): reference/forms.md
- Onboarding/setup guide composition: reference/onboarding.md
- Homepage composition (metrics, setup guide, callout cards): reference/app-home.md
- App icon, screenshots, feature graphic, listing copy (Register B, App Store listing): reference/marketing-surfaces.md

BFS also has non-design gates this file does not cover: minimum 50 net installs from active paid shops, minimum 5 reviews, a minimum recent rating, category-specific API requirements (discount functions, fulfillment tracking SLAs, subscription APIs, etc.), and the 100 KB theme-app-extension size limit. Those live in the BFS requirements page directly, not here.

## How to use this

Go through each numbered item below against the actual running app, embedded in a real Shopify Admin session (not a static screenshot). Do it at a typical desktop viewport and at a narrow/mobile viewport. Mark each item pass, fail, or needs-fix. Fix everything marked fail before submission; BFS review is manual and reviewers screenshot exactly the failure modes listed under each item.

## 0. Prerequisite gate: is the app even eligible to be evaluated on design?

- [ ] App is embedded in Shopify Admin using the latest version of App Bridge, loaded via a script tag in the `<head>` of every document (not a stale pinned version).
- [ ] Merchants can complete primary app workflows entirely inside Shopify Admin. The app does not embed external web pages as its primary UI.
- [ ] Any settings that control a connection to a third-party system (a linked social account, an external service) are manageable (connect/disconnect) from inside the embedded app, not only on an external site.

If any of these fail, fix them first. Everything below assumes the app is already a proper embedded admin surface.

## 1. Familiar (Section 4.1)

The test here: does this feel like a native Shopify admin surface, or does it feel like a web app wearing an iframe.

### 4.1.1 Follow UX best practices
This is the umbrella "good faith effort" requirement. Reviewers catch violations as concrete, screenshotted failures. Walk this literal list against the app:

| Check | Fail looks like |
|---|---|
| UI stability | Flickering, content repeatedly loading in/out, layout that shifts excessively as data arrives |
| Containers | Most content sits outside an `s-section` (or a bordered `s-box`). That section is the card-like container (see reference/layout-and-visual-design.md) |
| Button color | Primary buttons in green, purple, or any color that isn't the Polaris primary action color |
| Typography | A serif or script font used for the majority of content |
| Body text size | Noticeably different from Shopify Admin's own body text size |
| Background color | A background color that clearly departs from Admin (a black background is the canonical bad example) |
| Tabs | Clicking a tab changes content *above* the tabs, tabs wrap onto multiple lines, or tab position shifts between clicks |
| Icon consistency | Some items in a list/group have icons and others don't |
| Spacing | Spacing noticeably looser or tighter than Admin's (Admin runs on a 4px grid, see reference/layout-and-visual-design.md) |
| Contrast | Text fails basic WCAG 2.1 AA contrast (see reference/accessibility.md) |
| Back navigation | A sub-page has no back button to its parent |
| Mobile | The page requires full horizontal scrolling, or content is collapsed with no way to expand or reach it |
| Responsive breakpoints | A two-column desktop layout stays condensed into two columns on mobile instead of stacking |
| Navigation ownership | The app builds its own separate nav menu instead of integrating into the Shopify Admin nav menu |
| Parent highlighting | Navigating to a sub-page doesn't highlight the relevant parent nav item |
| Duplicate home link | A separate nav item exists that redirects to the homepage, in addition to the app name itself doing that job |
| Emoji in nav | Emoji rendered inside the Shopify Admin navigation menu |

### 4.1.2 Mobile-friendly
- [ ] Designed mobile-first, not mobile-adapted-as-an-afterthought (majority of storefront/admin-mobile traffic is mobile).
- [ ] No page requires horizontal scrolling to read in full.
- [ ] No content is permanently hidden or unreachable on a narrow viewport, only collapsed with a clear affordance to expand.
- [ ] Two-column and three-column desktop layouts stack into a single column on mobile.

### 4.1.3 Concise app name
- [ ] App name in the Admin nav does not truncate with an ellipsis, especially when the app is pinned.
- [ ] The app name contains no description ("Announcement Bar Pro for Upsells") — descriptions belong in the App Store listing (Register B, see reference/marketing-surfaces.md), not the nav-facing name.

### 4.1.4 Use the nav menu
- [ ] Primary navigation is integrated into the Shopify Admin nav menu via the App Bridge nav component (`s-app-nav` / referred to in some docs as the `ui-nav-menu` web component or `NavMenu` React component), not a custom in-app menu.
- [ ] Nav items are short, scannable nouns ("Templates", "Settings"), not verbs ("Manage Templates").
- [ ] Tabs are used sparingly, only for secondary navigation when the nav menu genuinely isn't sufficient. Same tab rules as above: never wrap, never move position, never change content above the tabs.

### 4.1.5 Use the contextual save bar
- [ ] Page forms, including a form inside `s-app-window`, save through the App Bridge Contextual Save Bar (`data-save-bar`), not a bespoke save button. An `s-modal` uses its action slots (4.1.6), not the save bar.
- [ ] Merchants can't navigate away from unsaved changes without hitting the CSB's Save or Discard action.
- [ ] No continuous/auto-save-as-you-type validation standing in for the CSB — this reads as "incongruous with standard Admin save UX" per the source doc (detail in reference/forms.md).

### 4.1.6 Use modals appropriately
- [ ] Large forms are not stuffed into max-height/max-width modals — they get a dedicated page instead (detail in reference/forms.md).
- [ ] Modals use `s-modal` with `heading` for the title and the `primary-action` / `secondary-actions` slots for CTAs, not the deprecated Polaris Fullscreen bar pattern.

## 2. Helpful (Section 4.2)

The test here: does the app actively help the merchant get their job done, or does it just sit there.

### 4.2.1 Spelling and grammar
- [ ] No spelling or grammar errors in headings, nav items, or CTAs.
- [ ] No unlabeled or unclear fields (the canonical bad example: a field just labeled "Time" with no unit).

### 4.2.2 Helpful onboarding
- [ ] Onboarding actually guides merchants to completion; it isn't decorative.
- [ ] Onboarding is concise and easy to find (not collapsed by default or scrolled out of view — see reference/onboarding.md for the 5-step-max, brief-and-direct guidance this enforces).
- [ ] Onboarding never implies installing a different app is a required step (a setup-guide primary button that installs another app is a rejection reason on its own).
- [ ] Onboarding doesn't ask for information without explaining why it's needed ("What types of products do you sell?" with no stated purpose is called out explicitly as a failure).
- [ ] There's a way to dismiss or remove the onboarding UI once it's complete — it doesn't linger forever.

### 4.2.3 Helpful homepage
- [ ] The homepage indicates whether the app is actually set up and working (for theme-attached apps, report app block/embed activation status via the `app.extensions()` API).
- [ ] Where relevant, the homepage surfaces real metrics/analytics (an email app that never shows open rates or recent campaign activity is the canonical failure).
- [ ] After a merchant dismisses every dismissible homepage element (banner, setup guide, callout card), the homepage still shows non-static, useful content — not just a handful of links or a static welcome message (full composition detail in reference/app-home.md).

### 4.2.4 Helpful error messages
- [ ] Error messages are red.
- [ ] Error messages appear next to (contextually near) the field or element they concern, not floated to the top of the page for a field-level problem.
- [ ] Error messages persist until resolved — they never auto-vanish on a timeout (a toast error disappearing after 5 seconds is the explicit failure case). This is different from a toast used for a transient success confirmation.
- [ ] Error copy guides the merchant toward a solution rather than just stating that something broke (deeper alert-pattern guidance in reference/alerts.md).

### 4.2.5 Guide merchants to logical actions
- [ ] In any group of related actions, the most logical action is the most visually dominant one (primary "Save changes" clearly outweighs secondary "Leave without saving" — never the reverse).
- [ ] Every screen has exactly one dominant primary action tied to the merchant's real next step, not two competing CTAs of equal weight.

### 4.2.6 Visible previews
- [ ] Any feature that lets merchants visually customize something (a theme block, an email template, a widget) shows a live, real-time preview.
- [ ] On desktop, the editor and its preview are both visible simultaneously — merchants should never have to toggle between them or scroll to see the effect of a change.

## 3. User-friendly: no dark patterns (Section 4.3)

The test here: read every screen and ask "does this manipulate, pressure, or mislead the merchant." If yes, it fails regardless of how polished it looks.

### 4.3.1 Don't make false claims
- [ ] No guaranteed/promised outcomes ("Double your revenue," "increase your sales by 18%").
- [ ] No promotion of another app (or your own other apps) with a star rating that doesn't match its real App Store rating.

### 4.3.2 Don't pressure merchants
- [ ] No visible, animated countdown timers pushing an upgrade (a 7-day-trial countdown is the named example).
- [ ] No guilt-trip CTA copy (forcing a click on "No thanks, I prefer less sales" to decline an upsell is the named example).
- [ ] No rewards offered in exchange for a 5-star review.

### 4.3.3 Don't distract merchants
- [ ] No modal or popover that auto-opens on page load, after a timer, or as a side effect of an unrelated action (a review-request modal popping up 3 seconds after load is the named failure).
- [ ] No animation used purely to draw attention unrelated to something the merchant just did (a wiggling "Upgrade to Pro" button is the named example). This is separate from the tasteful, `prefers-reduced-motion`-gated motion craft covered elsewhere — the line here is *unsolicited* attention-grabbing motion, not motion itself.
- [ ] No dramatic entrance animation on large elements (banners, cards) triggered by load, a timer, or an unrelated action.

### 4.3.4 Don't overwhelm merchants
- [ ] Complex forms are subdivided into logical field groupings rather than presented as one giant form (see reference/forms.md for the 5-input threshold that triggers sectioning).
- [ ] No two or more banners shown stacked/close together at once (top of page, or within one card).
- [ ] No large blocks of running prose replacing concise, scannable copy (see reference/content-and-voice.md).
- [ ] Red is reserved for error messaging or a destructive action — it is not used for anything else (see reference/layout-and-visual-design.md for the full semantic color-role table).

### 4.3.5 Don't impersonate Shopify
- [ ] The app icon can't be mistaken for a first-party Shopify icon (no similar gradient/background treatment). Full icon spec is Register B territory: reference/marketing-surfaces.md.
- [ ] The app doesn't use the Shopify Sidekick icon or a color resembling Shopify's "magic purple" to signal an AI feature.

### 4.3.6 Dismissible ads
- [ ] Any promotional/advertising content inside the app is dismissible by the merchant.
- [ ] Dismissed promotional content does not reappear later for that merchant (persist the dismissal — local storage or a database flag). Broader in-app promotion rules (light-touch branding, upgrade-gated visual states) are covered in reference/marketing-surfaces.md.

### 4.3.7 Label and disable premium features
- [ ] Plan-gated features are both visibly *and* functionally disabled for merchants who don't have access — never interactive-looking with the paywall sprung only at submission time, and never interactive with no visual indication it's gated.
- [ ] It's clear which specific tier/plan unlocks a specific feature when the app has multiple tiers — no ambiguity.
- [ ] Shopify Plus-exclusive features are hidden entirely from non-Plus merchants, not shown-then-blocked.

## 4. Performance and structural gates that show up as design failures

These are separate BFS sections (Performance, Distribution) but they surface as visible design/motion problems during a design review, so check them here too:

- [ ] Cumulative Layout Shift (CLS) at the 75th percentile is ≤ 0.1 — in practice this means loading skeletons that match the shape of the loaded layout, not content that jumps around as it arrives (matches the general "skeletons match the loaded shape" rule applied everywhere in this skill).
- [ ] Interaction to Next Paint (INP) at the 75th percentile is ≤ 200ms.
- [ ] If the app adds anything to the merchant's online store theme, it does so through theme app extensions, so uninstalling the app cleanly and automatically removes those theme blocks. This is a build/architecture concern more than a visual one, but a broken uninstall is a hard BFS rejection.

## 5. Ten-minute walkthrough script

Run this against the live app, in a real dev-store Admin session, before submitting:

1. Open the app cold. Time how long content takes to settle (CLS/INP feel). Watch for flicker or jank.
2. Look at the nav sidebar: is the app's primary nav integrated into Admin's own nav, not a custom menu? Click into a sub-page — does the parent nav item stay highlighted?
3. Resize the browser to a narrow/mobile width. Re-check every page for horizontal scroll, collapsed-and-unreachable content, and two-column layouts that fail to stack.
4. Open every form. Count inputs — more than five without sections is a fail. Make an edit and try to navigate away without saving: does the CSB intercept it?
5. Trigger an error (bad input, failed request). Confirm it's red, positioned next to the field, and doesn't disappear on its own.
6. Sit on the homepage after dismissing everything dismissible. Is there still real, non-static content?
7. If the app has a visual customization feature, confirm the live preview is visible simultaneously with the editor on desktop.
8. Hunt for dark patterns explicitly: any auto-opening modal, any countdown timer, any animation with no merchant-initiated trigger, any un-dismissible promo, any reward-for-review ask.
9. If the app has paid tiers, click through a gated feature as a merchant on a lower tier: is it clearly, functionally locked with a legible upgrade path?
10. Run contrast checks on any text that looked borderline gray-on-gray (Polaris "subdued" text is the most common trap here — see reference/accessibility.md).

Anything that fails gets fixed before submission, not noted as a "known issue." BFS review is manual, and reviewers screenshot exactly these failure modes.
