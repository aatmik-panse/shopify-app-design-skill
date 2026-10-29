---
name: shopify-app-design
description: Use when designing, redesigning, critiquing, or building the UI of a Shopify app — embedded admin apps built with Polaris s-* web components and App Bridge, App Home (the app's main landing page, homepage dashboard, Index/Details/Settings templates), Admin UI Extensions (action, selection action, block, configuration, print action, and runnable targets; admin.product-details.block.render and similar), onboarding and setup-guide flows, empty states, forms and settings pages, banners/toasts/inline alerts and loading/error states, navigation and information architecture (app nav, tabs, breadcrumbs, title bar), microcopy and voice/tone, accessibility (WCAG 2.1 AA, keyboard focus, screen readers), Built for Shopify design requirements and pre-submission review, and the App Store listing plus marketing assets (icon, screenshots, feature graphic, listing copy, landing page). Also use whenever a Shopify app's UI reads as generic, templated, "not a real Shopify app," or needs to feel more native and distinctive within Polaris constraints — covers layout composition, custom data visualization, empty-state illustration, and tasteful motion inside the token system. Not for non-Shopify frontend work (use impeccable or taste) and not for exhaustive component-prop lookup or codegen (hand off to the Polaris marketplace skills below).
---

# Shopify App Design

## Philosophy

This skill owns design judgment for Shopify apps: aesthetic direction, information architecture, content, UX pattern selection, accessibility, and Built-for-Shopify design quality. It is not a codegen tool and does not replace a search-then-validate component lookup.

The foundational fact to get right before anything else: Shopify apps live in two utterly different design registers.

- **Register A, Embedded Admin Surface** — the app's own pages inside Shopify Admin (App Home, settings, Admin UI Extensions). Visual truth = Polaris tokens + `s-*` components. No custom font, no brand-color takeover, no CSS fighting component internals. Creative freedom lives in IA, layout composition, state design, bespoke data viz, motion, and illustration — never in "our brand wearing an iframe."
- **Register B, Marketing Surface** — the App Store listing and any external marketing site. Full brand freedom applies here: typography, color, motion, a real visual identity. This is the one place a Shopify app should look like nothing else.

Bringing Register B into Register A is the single most common and costliest mistake in this domain. See `reference/philosophy-and-register.md` for the full treatment, including the inverse mistake (over-restraint on the listing) and why this distinction recurs in almost every design decision below.

## Non-negotiables (apply everywhere in Register A)

- WCAG 2.1 AA: body text ≥4.5:1 contrast, large text ≥3:1 — this is also a literal App Store review bar.
- Status communicated by text + tone together, never color alone. Visible keyboard focus on every interactive element. `prefers-reduced-motion` respected on all animation.
- One dominant primary action per screen, derived from real merchant state — never two CTAs with the same intent on one screen.
- Every fetch failure surfaces a banner with an inline Retry — never a silent fallback to a generic empty state.
- An empty state ≠ a filtered-to-zero state — the latter needs "clear filters," not onboarding copy.
- Copy must survive translation into longer languages (German runs ~30% longer than English) — don't design copy that only fits at English length.
- Loading skeletons match the shape of the loaded layout, not generic spinners.
- Most content sits in an `s-section` (or a bordered `s-box`). Don't nest sections for decoration. A heading or plain text needs no container.

## Routing table

| Task | Read | Why |
|---|---|---|
| Build/redesign the app's main embedded page (App Home) | `reference/app-home.md`, `reference/layout-and-visual-design.md`, `reference/accessibility.md` | Template choice (Homepage/Index/Details/Settings), Compositions (Setup guide, Metrics card, Empty state, Callout card), and the layout rules (single- vs two-column, containers, spacing) that make a homepage feel native instead of a generic dashboard collage — close with an accessibility pass (contrast, focus order, screen-reader labels) before calling it done |
| Build an Admin UI Extension (action/block/link/print/configuration) | `reference/admin-extensions.md`, `reference/component-patterns.md` | Target-type constraints differ sharply (blocks can't be hidden, only collapsed via `null`; bundle size caps; API-version-gated React vs `s-*` split) and the validator gotchas are non-obvious |
| Build a resource index/list page (search, filter, sort, bulk actions) | `reference/layout-and-visual-design.md`, `reference/component-patterns.md` | Index-table composition and `inlineSize="large"` for wide lists live in `layout-and-visual-design.md`; the `s-table` vs `s-grid` vs Index-table decision guide lives in `component-patterns.md` — a distinct task from the App Home row above, which is specifically about the homepage, not a sub-page list view |
| First-run / onboarding / setup-guide flows | `reference/onboarding.md` | Hard 5-step cap, dismissibility rules, progress-indicator pattern, and the empty-state-vs-onboarding distinction that's easy to blur |
| Forms / settings pages | `reference/forms.md`, `reference/accessibility.md` | Save Bar (not auto-save), sectioning threshold past 5 inputs, settings two-column layout, dedicated-page-not-modal rule for large forms — close with an accessibility pass on field labels, error announcements, and focus order |
| Banners / toasts / errors / loading states | `reference/alerts.md` | Banner vs Inline vs Toast selection depends on urgency/persistence; color-role semantics (red/yellow/orange/green) are fixed, not decorative |
| Navigation / information architecture | `reference/navigation.md` | App nav vs tabs vs breadcrumbs have strict, BFS-enforced rules (tabs never move content above them, app name owns the homepage link, 7-item nav truncation) |
| Microcopy / empty-state copy / voice and tone | `reference/content-and-voice.md` | Reading-level target, CTA verb-first convention, synonym consistency, translation-length safety |
| Accessibility audit (contrast, keyboard focus, screen readers, ARIA) | `reference/accessibility.md` | Concrete, testable bars — contrast ratios, focus order, ARIA patterns for tables/forms/modals — and a literal App Store review requirement per the non-negotiables above; read this whenever the task is scoped specifically to accessibility rather than folded into a broader page build |
| Pre-submission Built for Shopify design check | `reference/built-for-shopify-checklist.md` | Familiar/Helpful/User-friendly pillars and the concrete dark-pattern rejection reasons, distinct from the full app-store-review skill's broader compliance scope |
| App Store listing / marketing assets (Register B — different rules apply) | `reference/marketing-surfaces.md`, `reference/philosophy-and-register.md` | Icon/screenshot/feature-graphic specs and listing-copy rules live in full brand-freedom territory; do not carry Register A restraint into this file's guidance, and do not carry this file's freedom back into the app |
| "Make this feel less generic / more distinctive" | `reference/creative-playbook.md`, `reference/philosophy-and-register.md` | Names the Shopify-admin-specific slop tells (equal-weight KPI grids, vanity metrics, cosmetic setup guides, banner-blindness) and the "spend your boldness in one place" discipline for staying native while still standing out |

Most real tasks touch two or three rows — read all that apply before writing any markup.

## Process for any new page or flow

1. State a one-line design read: surface type (App Home vs Admin UI Extension vs Store listing), the merchant's real state, the one primary next action.
2. Sketch the section/layout plan in prose plus a small ASCII wireframe before writing any JSX.
3. Check the plan against `reference/creative-playbook.md` and the non-negotiables above.
4. Implement, validating any `s-*` markup before shipping (see Handoff).
5. Screenshot the real result in actual Shopify Admin at a typical viewport and a narrow one before calling it done.

## Handoff

This skill decides direction and reviews quality. For implementation:

- **shopify-polaris-app-home** — exhaustive `s-*` component/prop reference and codegen for the iframe-based App Home surface.
- **shopify-polaris-admin-extensions** — exhaustive component/prop reference and codegen for Admin UI Extensions.
- **shopify-app-store-review** — full pre-submission compliance check (broader than the design-only checklist here).

If those marketplace skills aren't installed, fall back to the `mcp__shopify-dev-mcp` tools directly: `search_docs_chunks` and `learn_shopify_api` for ground truth, and always run `validate_component_codeblocks` (or `validate_graphql_codeblocks` for GraphQL) on any generated `s-*` markup before handing it to a developer — prop names and enum values are easy to get subtly wrong, and this skill's reference files intentionally phrase uncertain specifics as "verify against current docs" rather than asserting them.
