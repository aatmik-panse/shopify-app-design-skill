# shopify-app-design

A [Claude Code](https://claude.com/claude-code) skill for designing, redesigning, critiquing, and building the UI of Shopify apps — embedded admin apps built with Polaris `s-*` web components and App Bridge.

Covers App Home, Admin UI Extensions, onboarding/setup-guide flows, forms and settings pages, banners/toasts/alerts, navigation and information architecture, content and voice, accessibility (WCAG 2.1 AA), Built for Shopify design requirements, and App Store marketing surfaces.

## Install

Clone into your Claude Code skills directory:

```bash
git clone https://github.com/aatmik-panse/shopify-app-design-skill.git ~/.claude/skills/shopify-app-design
```

Or copy just the folder contents into any project's `.claude/skills/shopify-app-design/` to scope it to that repo.

## Structure

- `SKILL.md` — entry point: philosophy, non-negotiables, and a routing table into the reference files below
- `reference/` — deep-dive docs per surface (App Home, admin extensions, forms, alerts, navigation, accessibility, content/voice, Built for Shopify checklist, marketing surfaces, creative playbook, philosophy and design register)

## Usage

Once installed, Claude Code picks this skill up automatically when a task touches Shopify app UI — designing a page, building an extension, reviewing onboarding flows, or auditing accessibility/Built-for-Shopify compliance. You can also invoke it explicitly:

```
/shopify-app-design
```
