# Philosophy and register

This file is the foundation the rest of this skill stands on. Every other reference file (layout, component-patterns, motion, copy, app-store-listing, and so on) assumes you've internalized the distinction below before it hands you a checklist. If you're only going to read one file before touching a Shopify app's design, read this one, produce the one-line design read it asks for, and only then move to the more tactical references.

## The two registers

A Shopify app lives in two places that could not be more different, design-wise. Confusing them is the single most common and most expensive mistake in Shopify app design, and it runs in both directions.

### Register A — Embedded Admin Surface

This is the app's own pages inside Shopify Admin: its App Home (the main landing page and any sub-pages), settings screens, and Admin UI Extensions (blocks, actions, print actions, configuration surfaces). The visual system here is Polaris design tokens plus the `s-*` Polaris web components, full stop.

What that means concretely:

- No custom font. The admin's type system is the type system.
- No arbitrary brand-color takeover. Color roles in the admin are semantically fixed (green = success, yellow = paused/caution, orange = in-progress/attention, red = blocked/error) and are not a palette to reskin with a brand's hex codes (shopify.dev/docs/apps/design/visual-design).
- No hand-rolled CSS that overrides component internals. The component system and its design tokens ARE the visual truth of this surface, not a shell to decorate.

None of that means Register A is creatively barren. Real creative freedom here lives in:

- **Information architecture and content strategy** — what belongs on the homepage, what's a settings page, what's an extension; how a workflow is broken into steps.
- **Layout composition** — grid and stack arrangement, section grouping, asymmetric spans, when to go single-column vs two-column vs dashboard-style equal-width, all within the token system (shopify.dev/docs/apps/design/layout).
- **State design** — the completeness and quality of empty, loading, error, and success states is where craft actually shows up in an admin surface.
- **Bespoke data visualization** — Polaris has no native chart components, so a well-designed custom SVG or canvas chart is a legitimate, real creative frontier, not a violation of the system.
- **Tasteful custom motion** authored in CSS around or inside component slots, always gated behind `prefers-reduced-motion`.
- **Illustration** for empty states and onboarding moments.
- **Sequencing and progressive disclosure** — the order in which information and choices are revealed.

The tone to aim for is "a first-party Shopify surface": native, restrained, unhurried, next-action-first. Not "our brand wearing an iframe." Built for Shopify's own design pillars back this up directly: apps should feel Familiar, Helpful, and User-friendly, and should generally look and behave like the rest of Shopify admin (shopify.dev/docs/apps/launch/built-for-shopify/requirements#design).

### Register B — Marketing Surface

This is the Shopify App Store listing (icon, screenshots, feature graphic, app card subtitle, listing copy) and any external marketing site or landing page for the app. Here, full brand creative freedom applies, completely. Distinctive typography, bold color strategy, custom layout, motion, a real visual identity. This is the one place a Shopify app can and should look like nothing else on the internet.

The App Store listing has its own hard rules, but they're compliance and honesty rules, not aesthetic restraint rules: images must primarily show actual app UI (no desktop chrome or browser windows in screenshots), each image must be unique, no pricing or stats or testimonials baked into images, no claims like "the first" or "the best" (shopify.dev/docs/api/polaris, App Store listing requirements 4.4 / 4.4.5 / 4.2 / 4.3). Within those honesty rules, go as bold as the brand deserves.

### The two costliest mistakes

**Bringing Register B into Register A.** Custom display fonts in the admin UI, marketing hero blocks at the top of App Home, saturated brand-color washes across cards, big "welcome to [App]!" banners for a returning user. This reads as "not a real Shopify app," damages merchant trust, and actively works against Built for Shopify eligibility (which explicitly penalizes button colors, fonts, and backgrounds that diverge from Shopify admin's own).

**Bringing Register A's restraint into Register B.** An App Store listing or marketing site so disciplined and Polaris-flavored that it fails to differentiate or sell the app at all. This is the inverse failure and it's just as real: a skill (or agent) that over-applies "be restrained, follow the system" as a universal law will produce a listing indistinguishable from every other app in the category, which defeats the entire purpose of a listing.

### Quick-reference table

| | Register A: Embedded Admin | Register B: Marketing Surface |
|---|---|---|
| Where | App Home, settings, Admin UI Extensions | App Store listing, marketing site |
| Visual system | Polaris tokens + `s-*` components, non-negotiable | Full brand system, your call |
| Typography | Admin's type system only | Custom display fonts welcome |
| Color | Semantic roles fixed; no brand-wash | Bold brand color strategy encouraged |
| Where creativity lives | IA, layout composition, state design, custom charts, motion-in-slots, illustration, sequencing | Everything: typography, color, layout, motion, voice |
| Failure mode if register is wrong | Feels like a foreign brand hijacking admin chrome; hurts trust and BFS eligibility | Feels generic, fails to sell the app, gets lost among competitors |
| Governing docs | shopify.dev App Design Guidelines, Built for Shopify requirements | App Store listing requirements (4.x), your own brand guidelines |

Component-level detail for Register A (exact `s-*` tags, props, gotchas) lives in reference/component-patterns.md. This file only owns which register you're in and why that matters.

## Read the room before you design anything

Before sketching a single section or writing a line of JSX, produce a one-line **design read**. This is not optional ceremony — it's the thing that prevents register confusion and prevents building the wrong screen for the wrong moment. State it explicitly, out loud, before anything else:

> **Design read:** [surface type] — merchant is in [real state] — primary next action is [one action].

Each of the three parts is load-bearing:

1. **Surface type.** Is this App Home (iframe or UI extension)? A settings page? An Admin UI Extension block/action/print-action? Or is this actually Register B — a listing screenshot, a feature graphic, a marketing page? Get this wrong and everything downstream is wrong, because the two registers have opposite rules.
2. **The merchant's real state.** Not an idealized persona — the actual condition of this specific screen right now. First install with zero data? Returning daily user with an established store? A filtered list that happens to return zero rows? An error mid-workflow? A merchant who dismissed the setup guide three days ago? State design (see the anti-slop list below) depends entirely on getting this right, and it's the single most common thing generic "AI-generated dashboard" output gets wrong: it designs for an idealized empty demo state and calls it done.
3. **The one primary next action.** What is the single most useful thing this merchant should do from this screen, given their real state? There should be exactly one dominant action. If you can't name it in one phrase, the screen doesn't have a clear job yet and needs more IA work before layout work.

Examples of a good design read:

- "App Home, iframe surface — merchant installed 40 minutes ago, has connected zero data sources, zero completed setup steps — primary next action is completing setup step 1 (connect a data source)."
- "Admin block on product-details page — merchant is viewing a product with an active AI-visibility scan running — primary next action is nothing to click, just status; the block should communicate progress honestly, no CTA competing with the product page's own primary actions."
- "App Store screenshot #3 — prospective merchant browsing the App Store comparing 4 competing apps — primary job is showing one specific, differentiating feature in under 2 seconds of attention."

Only after you've written that line do you move to a section/layout plan in prose plus a small ASCII wireframe, check it against the anti-slop list, and then implement. If you skip the design read, you'll produce output that's internally consistent but answers a question nobody asked (usually: "what does a generic SaaS dashboard look like," instead of "what does this merchant need right now").

## Writing your own app's product register

Every well-run design process for a specific app should have its own short, written product register: a one-page document the team (or the agent, on the team's behalf) can point back to instead of re-deriving taste from scratch on every screen. It answers the questions that generic Polaris compliance can't answer for you, because Polaris governs the how, not the why or the what.

The shape below is illustrative, not prescriptive of any particular product. Fill in the brackets for your own app.

```
USERS
Who actually uses this app day to day, and what do they already know?
(e.g. "solo merchants running 1-3 person Shopify stores, low technical
confidence, opening the app 2-3x a week between other admin tasks")

PRODUCT PURPOSE
One or two sentences, no jargon. What job does this app do for the
merchant that they'd otherwise have to do by hand or hire out?

BRAND PERSONALITY IN THREE WORDS
Pick exactly three adjectives and defend them. Not five, not one.
Three forces real trade-offs into the open.
(e.g. "calm, precise, encouraging" — vs. "playful, bold, irreverent" —
these two triples would produce visibly different copy and empty-state
illustration even inside the same Polaris constraints.)

ANTI-REFERENCES
Name 2-3 things this app should explicitly NOT feel like, and why.
This is often more useful than positive references, because it heads
off the default drift toward generic template output before it starts.
(e.g. "not a generic analytics dashboard with a wall of equal-weight
KPI tiles" / "not an upsell-heavy freemium app that nags on every
screen" / "not a marketing site that leaked into the admin")

DESIGN PRINCIPLES
3-5 short, specific, checkable rules that are true for THIS app,
derived from the purpose and personality above, not generic platitudes.
(e.g. "every number on the homepage has a decision attached to it or
it doesn't ship" / "one signature bespoke chart on the homepage, every
other surface stays plain Polaris" / "setup guide steps are gated to
real data state, never cosmetic checkboxes")

ACCESSIBILITY COMMITMENT
A specific, owned statement, not just "we follow WCAG." State the bar
and who it's for.
(e.g. "WCAG 2.1 AA across every custom-built surface including bespoke
charts and illustrations; status is always communicated by text and
tone together, never color alone, because a meaningful share of our
merchants use the admin with screen magnification or color vision
deficiency")
```

Two things make this exercise worth the ten minutes it takes:

- **The three-word personality constraint is doing real work.** Three words forces a team to actually choose and rank, rather than list every nice-sounding adjective. "Calm, precise, encouraging" and "playful, bold, irreverent" are both legitimate personalities for a Shopify app's Register B marketing site, but they'd produce different illustration styles, different empty-state copy, different motion pacing even though Register A's admin surface looks nearly identical either way (because Register A doesn't have much room for personality expression beyond copy, illustration, and the one signature moment per screen).
- **Anti-references prevent slop faster than positive references produce quality.** Telling an agent "be like Stripe" underdetermines the outcome. Telling it "not a generic KPI-card grid, not an upsell-heavy nag, not a marketing hero at the top of a returning-user's homepage" rules out the three most common failure modes before a single component gets placed.

Once written, this register is what "design read" statements and every downstream layout/copy/motion decision should be checked against. If a design decision can't be traced back to something in the register, that's a signal the register is incomplete or the decision is drifting.

## The anti-slop list (Register A)

Generic "AI dashboard" tells are not the same as generic "AI landing page" tells. A Shopify admin surface that reads as templated, low-effort, or AI-generated typically has one or more of these symptoms. Treat this as a checklist to run the layout plan against before implementation, every time:

- **A flat grid of equal-weight KPI cards with no hierarchy tied to what the merchant should actually do next.** This is a "SaaS dashboard collage," not a workflow tool. If every tile looks equally important, none of them are.
- **Vanity metrics with no decision attached.** A number that doesn't change what the merchant does next hasn't earned its pixels. Before placing any metric, name the decision it informs. If there isn't one, cut it or move it somewhere less prominent.
- **A marketing hero or welcome banner dominating the top fold for a returning user on a working surface.** That space belongs to live state and the next action, not to re-introducing the app to someone who already knows what it does. (This is exactly the Register B-into-A mistake described above, showing up in its most common concrete form.)
- **A "setup guide" that's cosmetic rather than genuinely sequenced and gated to real installation/data state.** Checkboxes that don't reflect actual completion, or steps presented as optional theater rather than real gating, undermine the merchant's trust in the rest of the UI.
- **Banners used for standing information that should live in a plain section instead.** Banner-blindness sets in fast; a banner that never goes away stops being read as urgent and starts being read as wallpaper.
- **Sections nested for decoration.** Most content sits in an `s-section` (or a bordered `s-box`). Nesting is for the heading outline, not another border. A heading or plain text needs no container.

Run this list against your section/layout plan before you write a single line of JSX, not after. Catching "this is just a KPI-card grid" at the prose-wireframe stage costs a rewrite of a sentence; catching it after implementation costs a rewrite of a component tree.

## Non-negotiables

See SKILL.md's Non-negotiables section — hold these in mind from the first sketch, not restated here.

## Handoff

This file owns design judgment: aesthetic direction, information architecture, content strategy, UX pattern selection, accessibility posture, the Built for Shopify design-quality bar, and creative craft within the Register A constraints. It is deliberately not a component API reference or a codegen tool.

- For exact `s-*` component names, props, and enum values: see reference/component-patterns.md, and validate any real markup with the `validate_component_codeblocks` MCP tool (or the `shopify-polaris-app-home` / `shopify-polaris-admin-extensions` marketplace skills if installed) before shipping.
- For Built for Shopify / App Store compliance self-review: use the `shopify-app-store-review` skill if installed.
- If no specialized skill is installed, fall back to the `mcp__shopify-dev-mcp` tools directly (`search_docs_chunks`, `learn_shopify_api`, `validate_component_codeblocks`, `validate_graphql_codeblocks`) rather than asserting an exact API surface from memory.
