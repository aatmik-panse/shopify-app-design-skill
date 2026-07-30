# Marketing Surfaces (Register B)

This file is **Register B**: the Shopify App Store listing (screenshots, feature graphic, video, listing copy, and — with a boundary-case caveat, see below — the icon) and any external marketing site or landing page for the app. It is the one surface in this entire skill where full brand creative freedom is not just allowed but expected. Everywhere else in this skill (App Home, Admin UI extensions, in-app settings, in-app promotional cards) is Register A: Polaris tokens and `s-*` components, no custom font, no brand-color takeover. Here, that constraint lifts.

If you take one thing from this file: **the app's own pages never get to look like a brand website, and the brand website never gets to look like a generic SaaS dashboard.** Confusing the two registers in either direction is the costliest mistake an app team makes.

## Where Register B actually lives, and where it doesn't

Register B applies to:

- The Shopify App Store listing: screenshots, feature graphic/video, listing copy.
- Any external marketing site or landing page for the app (not rendered inside Shopify admin).

Register B does **not** apply to:

- Anything rendered inside the app's own iframe or UI extensions, even when the content is promotional (a cross-sell card, an upgrade prompt, a "rate us" banner). That content still lives inside Shopify admin and still answers to Polaris and to Built for Shopify's anti-dark-pattern rules. See "The one exception" below, and see `reference/component-patterns.md` for the actual `s-*` markup of in-app compositions like the App card pattern.
- The app icon, cleanly. It doesn't sit fully in either register — see "The App icon" immediately below. This file is this skill's single canonical source for the icon's classification and spec; other reference files should link here rather than independently call it Register A or Register B.

## The App Store listing

### App icon

The icon is a boundary case, not a clean member of either register: it's the app's marketing identity in the App Store and on the Apps page, but it's also rendered small in the admin nav sidebar, so it can't be pure Register-B flourish. Treat this section as the canonical spec and classification for the icon across this skill — if another file needs to mention the icon, it should point back here rather than re-derive the rule. It has real geometric constraints (shopify.dev/docs/apps/design/visual-design):

- 1200px by 1200px, PNG or JPG, square, no rounded corners (Shopify auto-rounds it).
- Fill between 10/16ths and 12/16ths of the square (750px to 900px on a 1200px canvas) with the actual design.
- Leave a 1/16th margin (75px) around the edge free of visual elements.
- Bold colors, simple recognizable patterns.
- Avoid text in the icon; it becomes unreadable at small sizes.
- Avoid colors close to white or light gray; the icon renders on light backgrounds in the admin nav and can disappear.
- Never use the Shopify logo or any Shopify trademark, and never use an icon design you don't hold the rights to (shopify.dev/docs/api/polaris).

Within those constraints, the icon is real brand territory: color, shape language, a mark that's instantly recognizable at 24px in a sidebar and at full size in a Store search results grid. This is where "spend your boldness in one place" (see SKILL.md) applies to the identity system itself: get the icon distinctive, then let the rest of the listing support it.

### Screenshots

- 1600px by 900px (16:9), 3 to 6 desktop screenshots, at least one showing the app's actual UI (shopify.dev/docs/api/polaris).
- Crop out browser chrome and desktop backgrounds entirely. As of March 26, 2026 this is an enforced App Store requirement (4.4), not just a suggestion: screenshots must primarily show the app's real UI, and screenshots or feature images containing only the app logo are rejected.
- Each image in the listing must be unique (requirement 4.4.5): show different features, views, and states. Don't submit near-duplicates.
- No sensitive information or PII visible in any screenshot.
- No pricing, no reviews, no outcome guarantees baked into the image itself.
- Include mobile or POS screenshots if the app has that surface.
- Provide alt text for every image.

### Feature graphic and video

- Feature image: 1600px by 900px (16:9), one clear focal point, solid background with good contrast (4.5:1 recommended), alt text required.
- Don't use a Shopify logo in the feature image, and don't just repeat the app card subtitle as image text.
- If you have a video, make it a 2 to 3 minute promotional piece, not an instructional walkthrough; if you include screencasts, keep them to roughly 25% of the runtime. If there's no video, a static image conveying the app's core benefit is the fallback (shopify.dev/docs/api/polaris).

This is the one place a Shopify app can and should look like nothing else in the App Store. Full frontend-design distinctiveness practice applies directly here: deliberate typography pairing, a real color strategy instead of a safe default, purposeful motion in the video, an actual visual identity that differentiates the listing from the hundred other apps in the same category. Don't import Register A's restraint into this asset. A feature graphic that looks like a Polaris screenshot with a headline slapped on it is a wasted opportunity, not a safe choice.

## Listing copy voice

### Naming

- App Store listing name: unique, starts with the brand name (not a generic category descriptor), 30 characters or fewer. Example given by Shopify: "QTeck - Announcement Bar" is correct, "Announcement Bar - QTeck" is not. The `shopify.app.toml` config name should align with the listing name (shopify.dev/docs/api/polaris).
- This is a different, shorter constraint than the in-app nav name: the name shown in the admin sidebar truncates at 20 characters with an ellipsis, which is itself a rejection reason if it happens while pinned (shopify.dev/docs/apps/design/navigation). Don't assume the 30-character Store name and the 20-character nav name are the same string; write both deliberately.
- Don't put a description in the app name. Descriptions belong in the listing copy, not the name.

### App card subtitle (requirement 4.4.1)

Summarize the app concisely and explain its value. Don't:

- Stuff it with SEO keywords to game search ranking.
- Use personal merchant information without consent.
- Include data or statistics (put that on your own marketing site instead).

### App introduction

100 characters. Highlight the benefit and tie it to a measurable business outcome, in plain, complete sentences. Avoid keyword stuffing, data claims, incomplete sentences.

| Do | Don't |
|---|---|
| "We package and ship your orders. Fast, simple fulfillment can boost sales and delight customers." | "Get your products shipped fast. We'll take care of all the busy work for you." |

(shopify.dev/docs/api/polaris)

### Pricing (requirement 4.2)

Never put pricing information in images, including the icon, or anywhere in the listing outside the designated "Pricing details" section. Pricing text baked into a feature graphic or screenshot gets flagged.

### Truthfulness (requirement 4.3)

- No stats, data, or unsubstantiated superlative claims ("the first", "the best", "the only") in listing content or images.
- No reviews or testimonials in the listing or in images. Reviews are added automatically from merchant feedback; you don't get to curate quotes into your own assets.
- Disclose geographic or API-permission requirements if the app has them.
- Only claim language support you fully support in the live UI, not partial or planned translations.

### AI pre-submission scan (effective January 6, 2025)

Shopify runs an automated check before human review that flags: misuse of the app card subtitle, reviews or testimonials anywhere in the listing, pricing info outside the designated section, statistics or data display, links or bare URLs in listing copy, and generic (non-brand-led) app names. Screenshots are separately scanned for reviews/testimonials and stats/data overlays. Write the listing clean the first time; these are mechanical rejections, not judgment calls.

### Dark patterns that hold even in Register B

Full brand freedom in visual design does not mean full freedom in persuasion tactics. These bans apply to the App Store listing and any external marketing site, not just to in-app UI (shopify.dev/docs/apps/launch/built-for-shopify/requirements#design):

- Don't guarantee, promise, or strongly suggest a merchant outcome ("Double your revenue", "+18% sales").
- Don't display a star rating for another app that doesn't match its real App Store rating.
- Don't use fake reviews, false special offers, or countdown-timer urgency you don't mean.

## External marketing site or landing page

Same register, same freedom, same rules as the App Store listing copy above (truthfulness, no fabricated reviews, no pricing games). Beyond those guardrails, treat this exactly like any other marketing site: pick a real typographic pairing instead of the system default, commit to a deliberate color strategy instead of a safe neutral palette, use real motion, build an actual visual identity that would survive being shown next to the app's competitors with the logos removed. This is squarely general frontend-design territory, not Polaris territory. If you're building this site, apply the same distinctiveness discipline you'd bring to any landing page or product marketing site: nothing about Shopify's design system constrains it.

## The one exception: promotional content inside the app is still Register A

Anything the app renders about itself for marketing purposes (a "rate this app" ask, an upgrade prompt, a cross-sell for another app you make, a "what's new" callout) is still rendered inside Shopify admin. It doesn't get Register B's freedom just because its content is promotional. The Shopify admin is first and foremost a place merchants get work done, so in-app promotion has to be genuinely unobtrusive (shopify.dev/docs/apps/design/user-experience/marketing):

- Express brand with a light touch, mainly through imagery and illustration style, not through heavy customization. "Branding can set your app apart from others, but over-branding sets it apart from Shopify."
- Keep layout and common components (cards, spacing) consistent with the rest of Shopify admin. Don't skin them.
- Put promotional messages in a dismissible container (a card or gallery) at the bottom of the app homepage, or on a fully separate dedicated page. Once a merchant dismisses it, don't show it to that merchant again.
- Gate paid/premium features with a visually-disabled state plus subdued caption-style text explaining the upgrade requirement. Don't hide the feature and don't misrepresent its availability.
- No fake reviews, no false special offers, no countdown timers, no overselling. These erode trust in the app and in Shopify.
- Don't mix the page's primary call to action with an unrelated marketing or support ask; it obscures the actual next step. See SKILL.md's "one dominant primary action" rule, which this directly reinforces.
- Don't repeat the identical marketing message on every page of the app.
- For the actual implementation of a cross-promo card, use the App card composition pattern (see `reference/component-patterns.md`) rather than inventing custom markup.

Built for Shopify's design requirements repeat the same discipline in stricter, more mechanical terms for any in-app promotional surface: don't auto-show a promotional modal or popover on load, on a timer, or as a result of an unrelated action; don't dramatically animate a banner or card into view for attention; don't use animation to draw attention that isn't tied to something the merchant actually did; don't offer rewards for five-star reviews; don't use guilt-based CTA copy to discourage declining an upsell (shopify.dev/docs/apps/launch/built-for-shopify/requirements#design).

## Pre-submission checklist

- [ ] Icon: 1200x1200, square, no rounded corners, fill 750-900px, 75px margin, no text, no Shopify trademark, legible on light and dark backgrounds.
- [ ] Screenshots: 1600x900, 3-6 images, no browser chrome, no desktop background, each one unique, at least one real UI shot, alt text on all.
- [ ] Feature image/video: real focal point, no Shopify logo, no repeated subtitle text, video is promotional not instructional if included.
- [ ] Listing name: brand-first, under 30 characters, matches `shopify.app.toml`.
- [ ] Subtitle and introduction: no keywords stuffed in, no stats, no reviews, ties to one concrete benefit.
- [ ] Pricing text lives only in the Pricing details section, nowhere else, not in any image.
- [ ] No superlative claims, no testimonials, no fabricated ratings anywhere in listing content or images.
- [ ] Any in-app promotional content (cross-sell, upgrade prompt, review ask) reviewed against the Register A rules above, not this file's Register B freedom.
