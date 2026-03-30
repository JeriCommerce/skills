---
name: loyalty-template-creator
description: >
  Use when the user asks to "create a loyalty template", "build a loyalty page",
  "replicate a loyalty page style", "create a Shopify loyalty template",
  "make a loyalty page template", or needs to create a new CDN loyalty page
  template for a Shopify theme extension based on a reference website.
---

# Loyalty Template Creator

Create CDN loyalty page templates that replicate the visual style of a reference website, powered by the JeriCommerce loyalty-page JS engine.

**Output**: A self-contained `.liquid` file ready to be added as a Shopify theme section.

## Required Tools

This skill uses built-in tools only:

- `WebFetch` — Fetch reference pages and documentation
- `Read` — Read existing templates for reference
- `Write` — Create the template file
- `Bash` — Run lint and build (when in a Shopify theme repo)

## Workflow

### Step 0: Environment and documentation

Ask the user for the **template name**, **reference URL**, and **environment** (staging/production).

Use the CDN URL for the chosen environment:

| Environment | CDN URL |
|-------------|---------|
| **Staging** | `https://s.cdn.jericommerce.com/shopify/loyalty-page.js` |
| **Production** | `https://cdn.jericommerce.com/shopify/loyalty-page.js` |

Fetch these canonical docs using `WebFetch`. If anything in this skill conflicts with the live docs, **the docs win**:

1. **Template engine docs**: `https://support.jericommerce.com/storefront/loyalty-page-building-custom-templates-with-cdn-scripts/`
   - Covers: root element, data attributes, variable interpolation, earning/tier fields, loading states, section structure, icon system, CSS classes

2. **Widget routes docs**: `https://support.jericommerce.com/storefront/loyalty-page-setting-up-and-customizing-the-default-template/`
   - Covers: all `#jeri=` hash fragment routes for CTAs

### Step 1: Analyze the reference page

Use `WebFetch` on the reference URL and extract:

#### Visual Style
- Colors, fonts, layout, spacing, border-radius

#### Section Structure
- Hero, earnings, tiers, banners, CTAs

#### Color Palette
- Primary, accent, background, text, muted colors

#### Asset Inventory

Collect and list ALL reusable asset URLs found on the page:
- Hero/banner images (full URLs from `<img src>`, CSS `background-image`)
- Logo images
- Icons (earning icons, social icons, decorative)
- Tier badge images
- Background patterns or textures
- Any other visual assets

Present the asset inventory as a table:

| Asset | URL | Suggested use |
|-------|-----|---------------|
| Hero banner | `https://example.com/hero.jpg` | Hero section background |
| Gold badge | `https://example.com/gold.png` | Tier 3 badge via CSS `:nth-child()` |
| Instagram icon | `https://example.com/ig.svg` | Social earning icon |

**Ask the user** which assets to reuse directly (by URL) and which need replacements.

#### Font Detection
- Identify web fonts used (Google Fonts, Typekit, etc.) and their weights

#### When WebFetch Fails

If `WebFetch` fails (auth wall, SPA, etc.), ask the user for a screenshot or manual description.

### Step 2: Plan the template

Present the user with the proposed:
- Color palette
- Font stack
- Section layout
- Asset mapping
- Earning icon mapping
- Tier badge mapping
- CTA widget routes

Ask for confirmation before proceeding.

### Step 3: Generate the template

Generate a single self-contained `.liquid` file following all the rules below.

### Step 4: Verify

If working in a Shopify theme repo, run lint and build commands. Otherwise, remind the user to verify manually.

---

## Template Engine Reference

### Data Attributes

| Attribute | Purpose |
|-----------|---------|
| `id="jeri-loyalty-page-root"` | Required root element ID |
| `data-shop="{{ shop.permanent_domain }}"` | Shop identifier (Liquid variable) |
| `data-locale="{{ request.locale.iso_code }}"` | Locale for translations (Liquid variable) |
| `class="jeri-loyalty-page--loading"` | Loading state class — JS removes it automatically |
| `data-jeri="section.earnings"` | Earnings section wrapper — auto-hides if no earnings |
| `data-jeri="section.tiers"` | Tiers section wrapper — auto-hides if no tiers |
| `data-jeri="earning.type"` | Earning type element for conditional icon display |
| `data-jeri-show-if="<type>"` | Show element only for a specific earning type |
| `data-jeri-html="tier.content"` | Inject tier HTML content (tiers may contain HTML) |

### Variable Interpolation

Use `{variable}` syntax in **text nodes only** (never in HTML attributes):

#### Earning fields

| Variable | Description |
|----------|-------------|
| `{earning.title}` | Earning rule title |
| `{earning.description}` | Earning rule description |
| `{earning.points}` | Points awarded |

#### Tier fields

| Variable | Description |
|----------|-------------|
| `{tier.name}` | Tier name |
| `{tier.content}` | Tier description (may contain HTML — use `data-jeri-html`) |
| `{tier.pointsRequired}` | Points needed to reach tier |

### Earning Types

Each earning type gets a unique icon. Use `data-jeri="earning.type" data-jeri-show-if="<type>"` with a hardcoded `src` per image.

| Type | Description | Notes |
|------|-------------|-------|
| `purchase` | Points per purchase | Most common |
| `social` | Follow on social media | ALL social networks share this type |
| `referral` | Refer a friend | |
| `profile` | Complete user profile | |
| `verify` | Verify email/account | |
| `wallet` | Install wallet pass | |
| `scan` | Scan QR in store | |
| `link` | Click wallet link | |

#### Icon mapping pattern

```html
<!-- One img per earning type, only the matching one is shown -->
<img data-jeri="earning.type" data-jeri-show-if="purchase" src="https://example.com/purchase-icon.svg" alt="">
<img data-jeri="earning.type" data-jeri-show-if="social" src="https://example.com/social-icon.svg" alt="">
<img data-jeri="earning.type" data-jeri-show-if="referral" src="https://example.com/referral-icon.svg" alt="">
<!-- ...etc -->
```

### Tier Images

The JS engine does NOT inject tier images. Use CSS `:nth-child()` selectors with `content: url()` to assign images by position:

```css
.prefix__tier-badge:nth-child(1)::before { content: url('https://example.com/bronze.png'); }
.prefix__tier-badge:nth-child(2)::before { content: url('https://example.com/silver.png'); }
.prefix__tier-badge:nth-child(3)::before { content: url('https://example.com/gold.png'); }
```

### CTA Widget Routes

Use these `#jeri=` hash fragment routes for CTA buttons. For guests, redirect to the login page instead.

| Route | Action |
|-------|--------|
| `#jeri=loyalty/rewards` | Open rewards catalog |
| `#jeri=referrals` | Open referral program |
| `#jeri=loyalty` | Open loyalty landing |
| `#jeri=loyalty/earning-flows` | Open earning flows |

#### CTA pattern

```html
<!-- For logged-in customers -->
<a href="#jeri=loyalty/rewards">View Rewards</a>

<!-- For guests — use schema setting for redirect URL -->
<a href="{{ section.settings.guest_cta_url | default: '/account/login' }}">Sign In</a>
```

### Flatten Earnings Layout

Use `display: contents` on group/container wrappers to achieve a unified grid layout for earnings:

```css
.prefix__earnings-group { display: contents; }
```

---

## Template Structure Reference

### HTML Skeleton

Every template follows this structure:

```liquid
{%- comment -%} theme-check-disable RemoteAsset {%- endcomment -%}
<script src="https://s.cdn.jericommerce.com/shopify/loyalty-page.js" defer></script>
{%- comment -%} theme-check-enable RemoteAsset {%- endcomment -%}

<style>
  /* All CSS inline — template must be fully self-contained */
</style>

{%- comment -%} theme-check-disable RemoteAsset {%- endcomment -%}
<div id="jeri-loyalty-page-root" class="<prefix> jeri-loyalty-page--loading"
     data-shop="{{ shop.permanent_domain }}"
     data-locale="{{ request.locale.iso_code }}">

  <!-- Hero section (static Liquid — not JS-powered) -->

  <div class="<prefix>__content">
    <!-- Earnings section (JS-powered, data-jeri="section.earnings") -->
    <!-- Optional: static banners -->
    <!-- Tiers section (JS-powered, data-jeri="section.tiers") -->
    <!-- Optional: referral/promo banners -->
  </div>

  <!-- CTA footer -->

  <!-- Loading skeleton (visible while jeri-loyalty-page--loading is active) -->
</div>
{%- comment -%} theme-check-enable RemoteAsset {%- endcomment -%}

{% if section.settings.custom_css != blank %}
  <style>{{ section.settings.custom_css }}</style>
{% endif %}

{% schema %}
{
  "name": "Template Name",
  "settings": [...],
  "presets": [{ "name": "Template Name" }]
}
{% endschema %}
```

### Critical Implementation Rules

1. **Root element** — Must have `id="jeri-loyalty-page-root"`, `data-shop`, and `data-locale`
2. **Loading class** — Add `jeri-loyalty-page--loading` to root; JS removes it when ready
3. **Text interpolation** — `{earning.title}`, `{tier.name}`, etc. in TEXT NODES only (never in attributes)
4. **HTML content** — Use `data-jeri-html="tier.content"` for tier descriptions (may contain HTML)
5. **Custom icons** — Use `data-jeri="earning.type" data-jeri-show-if="<type>"` with hardcoded `src` per image
6. **Tier images** — Use CSS `:nth-child()` + `content: url()` (JS has no tier image injection)
7. **Flatten earnings** — Use `display: contents` on group/container wrappers for a unified grid
8. **Auto-hide** — `data-jeri="section.earnings"` and `data-jeri="section.tiers"` on section wrappers
9. **Schema name** — Max 25 characters. Must include `"presets"` array for "Add section" picker
10. **CDN script** — Use staging (`s.cdn.jericommerce.com`) or production (`cdn.jericommerce.com`) URL
11. **All CSS inline** — No external stylesheet dependency
12. **Theme-check comments** — Wrap `<script>` and root `<div>` with `theme-check-disable/enable RemoteAsset`

### Schema Rules (CRITICAL — must follow exactly)

The `{% schema %}` block defines theme editor settings. Shopify validates it strictly — any violation causes a `FileSaveError` on deploy.

#### Setting type rules

| Type | `default` allowed? | `default` format | Notes |
|------|-------------------|------------------|-------|
| `text` | Yes | String | `"default": "My Title"` |
| `textarea` | Yes | String | `"default": "Some text"` |
| `checkbox` | Yes | Boolean | `"default": true` |
| `number` | Yes | Number | `"default": 10` |
| `range` | Yes | Number | Must be within `min`/`max` |
| `select` | Yes | String | Must match one of `options[].value` |
| `color` | Yes | Hex string | `"default": "#0f0f0f"` |
| `url` | **NO** | — | **NEVER set `default` on `url` type settings.** Use Liquid fallback instead: `{{ section.settings.guest_cta_url \| default: '/account/login' }}` |
| `image_picker` | No | — | No default |
| `richtext` | Yes | HTML string | `"default": "<p>Text</p>"` |

#### Common schema errors and how to avoid them

| Error | Cause | Fix |
|-------|-------|-----|
| `setting with id="X" default must be a string or datasource access path` | `default` set on a `url` type setting | **Remove the `default` field entirely.** Use Liquid `\| default:` filter in the HTML instead |
| `Invalid schema: name must be 25 characters or less` | Schema `"name"` too long | Shorten to 25 chars max |
| `Invalid schema: presets must be an array` | Missing or malformed `presets` | Add `"presets": [{ "name": "..." }]` |
| `Invalid schema: setting default is invalid` | `default` value doesn't match type constraints | Check type rules above |

#### Correct schema pattern

```json
{
  "name": "Jeri Loyalty Brand",
  "class": "jeri-loyalty-brand-section",
  "settings": [
    {
      "type": "header",
      "content": "Hero"
    },
    {
      "type": "text",
      "id": "hero_title",
      "label": "Hero Title",
      "default": "Our Loyalty Program"
    },
    {
      "type": "textarea",
      "id": "hero_subtitle",
      "label": "Hero Subtitle",
      "default": "Join and start earning rewards today."
    },
    {
      "type": "text",
      "id": "guest_cta_text",
      "label": "Guest CTA Text",
      "default": "Join Now"
    },
    {
      "type": "url",
      "id": "guest_cta_url",
      "label": "Guest CTA URL",
      "info": "Where to redirect guests. Leave blank for /account/login"
    },
    {
      "type": "header",
      "content": "Sections"
    },
    {
      "type": "checkbox",
      "id": "show_earning_rules",
      "label": "Show Earnings Section",
      "default": true
    },
    {
      "type": "checkbox",
      "id": "show_tiers",
      "label": "Show Tiers Section",
      "default": true
    },
    {
      "type": "header",
      "content": "Logged-in CTA"
    },
    {
      "type": "text",
      "id": "logged_in_cta_label",
      "label": "Logged-in CTA Label",
      "default": "See Rewards",
      "info": "Opens the rewards widget when clicked."
    },
    {
      "type": "header",
      "content": "Colors"
    },
    {
      "type": "color",
      "id": "primary_color",
      "label": "Primary Color",
      "default": "#000000"
    },
    {
      "type": "color",
      "id": "accent_color",
      "label": "Accent Color",
      "default": "#FFFF42"
    },
    {
      "type": "header",
      "content": "Advanced"
    },
    {
      "type": "textarea",
      "id": "custom_css",
      "label": "Custom CSS"
    }
  ],
  "presets": [
    {
      "name": "Jeri Loyalty Brand"
    }
  ]
}
```

#### URL fallback pattern in HTML

Since `url` settings cannot have defaults, always use Liquid `| default:` in the template HTML:

```liquid
<a href="{{ section.settings.guest_cta_url | default: '/account/login' }}">
  {{ section.settings.guest_cta_text | default: 'Join Now' }}
</a>
```

### BEM Naming

Use a unique prefix per template in `kebab-case`:

```css
.siwon-copy { }              /* block = root */
.siwon-copy__hero { }        /* element */
.siwon-copy__hero--dark { }  /* modifier */
```

- Prefix = template name (e.g., `.mystore-`, `.brand-name-`)
- Add the prefix as a class on the root element alongside `jeri-loyalty-page--loading`
- All CSS selectors scoped under this prefix to avoid conflicts

### Responsive Design

Include mobile breakpoints:

```css
@media (max-width: 768px) {
  /* Tablet adjustments */
}

@media (max-width: 480px) {
  /* Mobile adjustments */
}
```

Key responsive patterns:
- Grid columns collapse (3 → 2 → 1)
- Font sizes reduce
- Padding/margins shrink
- Hero images may change aspect ratio or hide
- Tier cards stack vertically

### Loading Skeleton

Include a skeleton UI visible only while `jeri-loyalty-page--loading` is active:

```css
.jeri-loyalty-page--loading .prefix__content { display: none; }
.jeri-loyalty-page--loading .prefix__skeleton { display: block; }
.prefix__skeleton { display: none; } /* hidden once loaded */
```

### Example Template

For a complete working example with custom icons, tier images, static banners, and all engine features, see `siwon-copy.liquid` in the `shopify-extensions` repo:

```
shopify-extensions/theme-extensions/loyalty-page/templates/siwon-copy.liquid
```
