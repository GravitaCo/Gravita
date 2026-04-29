# Gravita — Technical Stack & Standards

## Core Tools

| Tool | Purpose |
|------|---------|
| **Webflow** | Website design, development, and hosting. Primary delivery platform. |
| **Figma** | UI/UX design, component design, design system source of truth. |
| **Slack / Teams** | Client communication (client preference). |
| **GSAP** | Scroll-triggered animations, text animations, custom interactions. |
<!-- TODO: Benn — add other tools in your stack (project management, analytics, etc.) -->

## Webflow Sites (Gravita Workspace)

Current sites accessible via Webflow MCP:

| Site | Domain | Purpose |
|------|--------|---------|
| Gravita-New | gravita.co | Live production site |
| Gravita Framework & Design System | gravita-ds.webflow.io | Internal design system reference |
| Gravita 2026 | (staging) | New version in progress |

## Webflow Standards

<!-- TODO: Benn — fill in your Webflow conventions -->
<!-- Suggested areas: -->
<!-- - Naming conventions for classes (BEM? Client Specific? Utility-based?) -->
<!-- - CMS collection structure patterns -->
<!-- - Component / symbol naming conventions -->
<!-- - Page structure conventions -->
<!-- - SEO defaults (meta titles, descriptions, OG images) -->
<!-- - Accessibility requirements (ARIA, contrast, keyboard nav) -->
<!-- - Responsive breakpoint strategy -->

## Design System Approach

Gravita builds design systems that give clients **autonomy** — the ability to ship compliant, accessible pages without waiting on a designer or developer.

<!-- TODO: Benn — expand on: -->
<!-- - How you structure design systems in Figma (components, variants, tokens) -->
<!-- - How that maps to Webflow (symbols, classes, CMS) -->
<!-- - What "compliant and accessible" means in practice (WCAG level, audit process) -->
<!-- - How you hand off the design system to clients -->

## Frontend Development Standards

### JavaScript
- GSAP for scroll-triggered animations (data-attribute driven, webkit prefix support)
- jQuery used selectively (e.g., CMS search timing fixes)
- Custom modal patterns (ARIA attributes, CSS display state, data-modal-target)
- Marquee animations with mobile-specific speed adjustments
- Storylane embed handling (data-src autoplay workaround)

### CSS
- Hover states for image cards
- Anchor scroll offset overrides for Webflow's default JS behavior
- Mobile-first responsive approach

<!-- TODO: Benn — add any coding standards or patterns you want agents to follow -->

## SEO & Accessibility Standards

<!-- TODO: Benn — fill in baseline requirements -->
<!-- - Target WCAG level (AA?) -->
<!-- - SEO checklist for new pages -->
<!-- - Structured data / schema markup conventions -->
<!-- - Image optimization requirements -->
<!-- - Core Web Vitals targets -->

## Integrations

Common third-party integrations Gravita works with:

<!-- TODO: Benn — list common integrations across client projects -->
<!-- Examples: HubSpot, Google Analytics, Google Tag Manager, LinkedIn, etc. -->
