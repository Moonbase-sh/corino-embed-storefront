# Corino

A single-page marketing site for **Corino Studio**, a fictional Oslo-based audio-plugin company. It sells two plugins — **HALO** (saturator) and **DRIFT** (granular reverb) — plus a bundle, and is wired end-to-end to a real [Moonbase](https://moonbase.sh) tenant for auth, cart, checkout, downloads, and account management.

This repo exists as a **reference implementation**: the smallest plausible site (one HTML file, no build tooling) that still demonstrates how to embed the Moonbase storefront with realistic intent links, conditional rendering, and live-bound pricing.

## Stack

- Plain HTML5 + CSS + vanilla JS in a single `index.html`
- Google Fonts (Space Grotesk, JetBrains Mono, Inter)
- [`@moonbase.sh/storefront`](https://www.npmjs.com/package/@moonbase.sh/storefront) loaded directly from `assets.moonbase.sh`
- No bundler, no package.json, no framework, no build step

## Run it locally

The site needs to be served over HTTP — `file://` won't work because the Moonbase module is loaded as an ES module and talks to a real tenant. Any static server will do:

```bash
python -m http.server 8000
# or
npx http-server -p 8000
```

Then open <http://localhost:8000>.

## Layout

```
.
├── index.html      # the entire site — markup, styles, scripts
├── assets/         # product artwork (SVG + PNG)
└── .context/       # gitignored scratch space for agents
```

## How Moonbase is integrated

The integration has three moving parts: one initialization call, intent links for user actions, and `data-moonbase-*` attributes for conditional UI and live data binding. All three live inside `index.html`.

### 1. Initialization

At the bottom of the page:

```html
<script type="module" src="https://assets.moonbase.sh/storefront/moonbase.js"></script>
<script type="text/javascript">
  document.addEventListener('DOMContentLoaded', () => {
    Moonbase.setup('https://corino-demo.moonbase.sh', {
      toolbar: { enabled: false },
      theme: {
        dark: true,
        colors: { primary: '#eca45d' },
        corners: 'sharp',
        buttons: 'light',
        fonts: { heading: 'Montserrat', body: 'Inter' },
      },
    });
  });
</script>
```

- **Tenant**: `https://corino-demo.moonbase.sh` — the Corino dedicated demo tenant.
- **Toolbar**: disabled (we drive everything from page UI instead).
- **Theme**: dark, sharp corners, golden primary `#eca45d`, light buttons, custom heading/body fonts. The theme only restyles Moonbase-rendered surfaces (modals, forms, checkout); the page's own CSS is untouched.

### 2. Intent links

Moonbase exposes deep links via the `?mb_intent=…` query parameter. Plain `<a href>` is enough — no JS handlers needed.

| Action | Example |
| --- | --- |
| Sign in | `?mb_intent=sign_in` |
| View account | `?mb_intent=view_account` |
| View cart | `?mb_intent=view_cart` |
| Add product to cart | `?mb_intent=add_to_cart&mb_product_id=halo` |
| Add bundle to cart | `?mb_intent=add_to_cart&mb_bundle_id=duo-bundle` |
| View product (trial) | `?mb_intent=view_product&mb_product_id=drift` |
| Download owned product | `?mb_intent=download_product&mb_product_id=halo` |

The demo tenant uses three IDs: `halo`, `drift`, and `duo-bundle`.

### 3. Declarative attributes

Moonbase observes two attributes on regular HTML elements:

**`data-moonbase-if="…"`** — conditional rendering. Examples used in the site:

- `!user` / `user` — toggle the **Log in** vs **My account** links in the nav.
- `cart.has_items` — show the cart-count badge only when there's something in it.
- `product.halo.owned` / `!product.halo.owned` — swap the product card CTA between **Download** and **Add to cart**.
- `product.halo.has_discount` — show the strike-through original price and the discount badge.
- `bundle.duo-bundle.has_discount` — show the bundle discount row.

**`data-moonbase-render="…"`** — bind text content to live data:

- `user.name` — greet the logged-in user.
- `cart.item_count` — populate the cart badge.
- `product.<id>.price`, `product.<id>.original_price`, `product.<id>.discount_name` — live pricing on each product card.
- `bundle.<id>.price`, `bundle.<id>.discount_total` — bundle pricing.

The pattern is: write static, sensible-looking markup for the empty state, then let Moonbase hydrate the text and visibility once it knows about the user, cart, and catalog.

## Further reading

- [Moonbase embedded storefront docs](https://moonbase.sh/docs/storefronts/embedded/)
