---
name: loyalty-template-creator
description: >
  Use when the user asks to "create a loyalty template", "build a loyalty page",
  "replicate a loyalty page style", "create a Shopify loyalty template",
  "make a loyalty page template", or needs a custom JeriCommerce loyalty page
  for a Shopify theme based on a reference website. Builds a Shopify section
  with the JeriCommerce Storefront SDK.
---

# Loyalty Template Creator

Create a Shopify section that replicates the visual style of a reference website and shows the merchant's JeriCommerce loyalty program with the Storefront SDK (`window.JeriCommerce`).

**Output**: A self-contained `.liquid` section (markup, inline CSS, one module script and a schema) ready to add to a Shopify theme.

## Workflow

### Step 0: Read the SDK documentation

Ask the user for the **template name** (used as the section filename) and the **reference URL**.

Fetch these canonical guides. If anything in this skill conflicts with them, **the guides win**:

1. **Storefront SDK**: `https://support.jericommerce.com/storefront/storefront-sdk/` (setup, session, behavior after an action, events, errors)
2. **SDK reference**: `https://support.jericommerce.com/storefront/storefront-sdk-reference/` (every method and the fields of each result)
3. **Custom loyalty page**: `https://support.jericommerce.com/storefront/storefront-sdk-custom-loyalty-page/` (complete working template to start from)
4. **Storefront widget**: `https://support.jericommerce.com/storefront/how-to-use-the-jericommerce-storefront-widget/` (widget routes for CTAs)

`https://support.jericommerce.com/llms-full.txt` has the full text of every guide in Markdown if a page cannot be fetched.

### Step 1: Analyze the reference page

Inspect the reference URL and extract:

- Visual style: colors, fonts, layout, spacing, border radius
- Section structure: hero, rewards, ways to earn, tiers, banners, CTAs
- Image and asset URLs: hero images, icons, banners, tier badges
- Color palette: primary, accent, background, text, muted

If the page cannot be fetched because of auth, SPA behavior or blocking, ask the user for a screenshot or equivalent reference material.

### Step 2: Plan the template

Present the user with:

- Proposed color palette and font
- Section layout, for visitors and for signed-in customers
- Which SDK data each section shows (balance, tier progress, rewards, discount codes, ways to earn, points history, tiers, wallet card)
- Static banners, and which CTAs open the widget

Ask for confirmation before proceeding.

### Step 3: Generate the template

Start from the template in the custom loyalty page guide and restyle it. Keep one `<script type="module">`, inline `<style>` and a `{% schema %}`.

#### Critical implementation rules

1. Check `window.JeriCommerce` first. When it is missing, the **SDK** app embed is off: show a short notice instead of an empty page.
2. Get the instance with `await window.JeriCommerce.ready()`. Never call the JeriCommerce API with `fetch`, and never handle tokens, emails or customer ids: the SDK acts for the customer signed in to Shopify.
3. Decide the view with `jeri.auth.getSession()`. Visitors get public data (`rewards.list({ scope: "public" })`, `earnings.list()`, `tiers.list()`) and a sign-in button that calls `jeri.auth.login()`. Customers get their own data.
4. Only use methods and fields listed in the SDK reference. Do not invent fields.
5. Show an action's result from its promise (for example `claimed.code` from `rewards.claim()`), and keep the page up to date with `jeri.on("customer:updated" | "rewards:updated" | "discounts:updated", ...)`. Never call `refresh()` after an action.
6. Handle errors by `error.code`, and by `error.serverCode` for `api_error` (for example `NOT_ENOUGH_POINTS`). Never parse message text. Re-enable a button after a failure.
7. Ways to earn: render `programEngagementFlows[0]` only when `active`. Use `earnings.execute({ earningId, action: "flow" })` when `properties.executeOnClick` is true, and `action: "click"` with the network key as `linkId` for **Follow us** links (open the social page yourself).
8. **Points as cash** rewards (`type: "dynamic"`) need `pointsToRedeem` between `minPoints` and `maxPoints`.
9. Build DOM with `textContent`. Only a tier's `content` is HTML written by the merchant.
10. Scope every selector to the section root and use a unique BEM prefix, e.g. `.brand-`.
11. All CSS inline; no external stylesheets or scripts. Include responsive breakpoints for `768px` and `480px`.
12. Schema name 25 characters or fewer, with a `presets` array so the section appears in **Add section**. Put merchant copy and images in settings.

#### Widget CTAs

Open the widget with `jeri.widget.open({ page })` or a `#jeri=` link:

| `page` | Route | Opens |
|---|---|---|
| `rewards` | `#jeri=loyalty/rewards` | Rewards catalog |
| `loyalty` | `#jeri=loyalty/earning-flows` | Ways to earn |
| `referrals` | `#jeri=referrals` | Referral program |
| `coupons` | `#jeri=profile/coupons` | Discount codes |
| `profile` | `#jeri=profile` | Profile |

`widget.open()` rejects with `widget_unavailable` when the merchant turned the widget off. Fall back to a page of the store.

### Step 4: Install and verify

Tell the user how to install it, as in the custom loyalty page guide:

1. Turn on the **SDK** app embed: **Online Store** > **Themes** > **Customize** > **App embeds**.
2. Add the file as `sections/<name>.liquid`, create a page template that uses it, and create a page with that template.
3. Check the page signed out and signed in with a test customer: data renders, claiming a reward shows its code and updates the balance, and errors show a message.

If you are working inside a theme repository, run its linter (for example `shopify theme check`) before finishing.
