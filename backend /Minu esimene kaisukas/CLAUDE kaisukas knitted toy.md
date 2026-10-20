# CLAUDE.md — "Minu esimene kaisukas" webshop

Rules and context for Claude when working in this repository. Read this file fully before making changes.

## 1. Project overview

- **Shop name:** Minu esimene kaisukas ("My first cuddly toy")
- **What it sells:** handmade knitted toys (kootud mänguasjad)
- **Market:** Estonia
- **Site language:** Estonian (`et`). All user-facing text MUST be in Estonian.
- **Scope:** front-end only (React). No deployment — do not add CI/CD, hosting configs, Dockerfiles or deploy scripts unless explicitly asked.
- **Core features:**
  1. Home page with a product carousel of exactly **10 product cards**
  2. Product detail view
  3. Shopping cart
  4. Checkout with **parcel locker (pakiautomaat) delivery**
  5. **Card payment** (pangakaart)

## 2. Tech stack

- React 18+ with **TypeScript** (strict mode)
- **Vite** as the build tool
- **React Router** for routing
- Styling: **CSS Modules** + global CSS variables (design tokens in `src/styles/tokens.css`). No Tailwind, no UI kits unless asked.
- State: React Context + `useReducer` for the cart; persist cart to `localStorage`
- Forms: controlled components or `react-hook-form`; validation with `zod`
- Carousel: build a small custom accessible carousel, or use `embla-carousel-react` (preferred if a library is needed). Do not use heavy/jQuery-based sliders.
- Testing: Vitest + React Testing Library
- Linting/formatting: ESLint + Prettier

Do not add new dependencies without a clear reason; mention every new dependency in your summary.

## 3. Folder structure

```
src/
  assets/            # logo.svg, images, icons
  components/        # reusable UI (Button, ProductCard, Carousel, Header, Footer...)
  features/
    cart/            # CartContext, cartReducer, CartDrawer
    checkout/        # CheckoutForm, ParcelLockerSelect, PaymentForm
    products/        # ProductCarousel, ProductDetail
  data/
    products.ts      # the 10 products (mock data)
    parcelLockers.ts # mock locker list
  i18n/
    et.ts            # ALL Estonian UI strings live here
  pages/             # Home, Product, Cart, Checkout, OrderConfirmation, NotFound
  services/          # api.ts, payment.ts, shipping.ts (mock-able adapters)
  styles/
    tokens.css       # colors, spacing, radii, shadows, fonts
    global.css
  types/             # Product, CartItem, Order, ParcelLocker...
  App.tsx
  main.tsx
```

## 4. Brand & logo

- Name is always written: **Minu esimene kaisukas** (only first letter capitalised).
- Logo: `src/assets/logo.svg` — a simple, soft line-drawn knitted teddy/bunny with a small heart or yarn ball, plus the wordmark "Minu esimene kaisukas" in a rounded serif or handwritten-style font. Lilac on cream/white.
- Logo must work at 32px (favicon) and in the header (~48–64px height). Provide `favicon.svg`.
- Always add `alt="Minu esimene kaisukas"` to the logo image.

## 5. Visual design — lilac, aesthetic style

Soft, calm, handmade, cosy. Think pastel, rounded shapes, lots of whitespace, gentle shadows, subtle knit/yarn texture details. Never loud, never neon, no harsh black.

### Design tokens (`src/styles/tokens.css`)

```css
:root {
  /* Lilac palette */
  --color-lilac-50:  #F8F4FC;
  --color-lilac-100: #EFE6F8;
  --color-lilac-200: #E0CFF1;
  --color-lilac-300: #CDB4E6;  /* main accent */
  --color-lilac-500: #A383C9;
  --color-lilac-700: #6E4F94;  /* buttons, links (passes contrast on white) */
  --color-lilac-900: #3F2A5C;  /* headings, body text */

  --color-cream: #FFFBF7;
  --color-blush: #F6DDE3;      /* small secondary accent */
  --color-sage:  #DCE8DC;      /* success states */
  --color-error: #B94A5A;

  --font-heading: "Fraunces", "Playfair Display", Georgia, serif;
  --font-body: "Nunito", "Quicksand", system-ui, sans-serif;

  --radius-sm: 8px;
  --radius-md: 16px;
  --radius-lg: 28px;
  --radius-pill: 999px;

  --shadow-soft: 0 6px 24px rgba(110, 79, 148, 0.12);
  --shadow-hover: 0 10px 32px rgba(110, 79, 148, 0.2);

  --space-1: 4px; --space-2: 8px; --space-3: 12px;
  --space-4: 16px; --space-6: 24px; --space-8: 32px; --space-12: 48px;

  --transition: 200ms ease;
}
```

### Rules
- Background: `--color-cream` or `--color-lilac-50`. Text: `--color-lilac-900`.
- Primary buttons: `--color-lilac-700` background, white text, pill radius.
- Text and interactive elements must meet **WCAG AA contrast** (4.5:1 for body text). Light lilac shades are for backgrounds/decoration only, never for body text.
- Images: soft rounded corners (`--radius-md`), product photos on light lilac backgrounds.
- Animations: subtle only (fade, gentle lift on hover). Respect `prefers-reduced-motion`.
- Always use tokens — no hard-coded hex colors in components.
- Mobile-first responsive design. Breakpoints: 480px, 768px, 1024px, 1280px.

## 6. Product carousel — exactly 10 cards

- Home page shows **one carousel with exactly 10 product cards**. Not more, not fewer. Do not add grids, extra product sections, "load more" or pagination.
- Product data lives in `src/data/products.ts` as an array of exactly 10 items. Add a test that asserts `products.length === 10`.
- Visible cards per view: 1 (mobile) · 2 (≥768px) · 3 (≥1024px) · 4 (≥1280px).
- Controls: previous/next arrow buttons + dot indicators; swipe on touch; keyboard arrows when focused.
- Accessibility: `role="region"` + `aria-roledescription="karussell"` + `aria-label="Tooted"`; each slide `aria-label="Toode 3 / 10"`; buttons labelled "Eelmine" / "Järgmine". No autoplay (or autoplay only with a visible pause button and paused on hover/focus).

### ProductCard contents
- Photo (lazy loaded, with Estonian `alt` text)
- Name (e.g. "Lilla jänku Liisu")
- Short description (1 line)
- Price in euros, Estonian format: `24,90 €` (use `Intl.NumberFormat('et-EE', { style: 'currency', currency: 'EUR' })`)
- Button: "Lisa ostukorvi"
- Optional badge: "Uus", "Käsitöö", "Viimane eksemplar"

### Product type
```ts
export interface Product {
  id: string;
  slug: string;
  name: string;           // Estonian
  shortDescription: string;
  description: string;
  priceCents: number;     // always store money in cents
  images: string[];
  material: string;       // e.g. "100% puuvillane lõng"
  sizeCm: number;
  ageRecommendation: string; // e.g. "Sobib alates sünnist"
  inStock: boolean;
  badge?: 'Uus' | 'Käsitöö' | 'Viimane eksemplar';
}
```

Example product names (keep this cosy tone): Lilla jänku Liisu, Karu Mõmmi, Elevant Ella, Kiisu Miisu, Lammas Lumi, Rebane Rebeka, Kilpkonn Kalle, Siil Siiri, Ükssarvik Unistus, Kutsu Muri.

## 7. Estonian language rules

- `<html lang="et">`.
- Every UI string goes into `src/i18n/et.ts` — no hard-coded strings inside components.
- Use correct Estonian characters: **õ, ä, ö, ü, š, ž**. Never replace them with plain letters.
- Formats: prices `24,90 €` · dates `20.10.2026` · phone `+372 5123 4567`.
- Tone: warm, friendly, simple; address the customer with polite "Teie"/"te" in checkout and legal text, may be softer on marketing copy.
- Key vocabulary:

| English | Estonian |
|---|---|
| Add to cart | Lisa ostukorvi |
| Cart | Ostukorv |
| Checkout | Vormista tellimus |
| Pay | Maksa |
| Parcel locker | Pakiautomaat |
| Choose parcel locker | Vali pakiautomaat |
| Shipping | Tarne |
| Total | Kokku |
| incl. VAT | sh käibemaks |
| Card payment | Kaardimakse |
| Card number | Kaardi number |
| Expiry date | Kehtivusaeg |
| Name | Nimi |
| E-mail | E-post |
| Phone | Telefon |
| Order confirmed | Tellimus on kinnitatud |
| Thank you! | Aitäh! |
| Out of stock | Otsas |
| Terms and conditions | Müügitingimused |
| Privacy policy | Privaatsuspoliitika |

## 8. Cart

- Cart icon in header with item count badge.
- Change quantity, remove item, show subtotal, shipping, total.
- Persist in `localStorage` (key: `kaisukas-cart`).
- All money math in **integer cents**. Never use floats for prices.

## 9. Checkout — parcel locker delivery

Delivery method is **parcel locker only** (pakiautomaat).

- Carrier choice (radio buttons with logos/text): **Omniva**, **DPD Pickup**, **SmartPosti** (formerly Itella).
- After selecting a carrier, the user picks a locker from a **searchable dropdown** (search by city or locker name, grouped by city).
- Locker data comes through `src/services/shipping.ts`. For now use mock data in `src/data/parcelLockers.ts`; keep an adapter interface so real carrier APIs (e.g. Omniva public locations JSON) can be plugged in later, ideally via a backend proxy.
- Show shipping price per carrier (mock prices, e.g. 3,49 €). Optional free shipping threshold (e.g. from 50 €) — configurable constant.
- Required fields: Nimi, E-post, Telefon (validate Estonian mobile format, `+372` prefix allowed), selected locker.
- Checkout steps: 1) Kontaktandmed → 2) Tarne (pakiautomaat) → 3) Makse → Kinnitus.

```ts
export interface ParcelLocker {
  id: string;
  carrier: 'omniva' | 'dpd' | 'smartposti';
  name: string;
  address: string;
  city: string;
  postalCode: string;
}
```

## 10. Payment — card payment

- Payment method: **card payment** (Visa, Mastercard).
- **Never** collect, store, log or send raw card data through our own code or state. Use the payment provider's hosted fields / hosted page (e.g. **Stripe Elements**, or Estonian providers such as **Montonio** or **EveryPay**).
- Wrap the provider in `src/services/payment.ts` with an interface (`createPayment`, `confirmPayment`) so the provider can be swapped.
- Until a backend exists, implement a **mock payment provider** (clearly marked `// MOCK – replace before production`) that simulates success/failure; show a visual card form only as a demo UI that does not keep card numbers in state beyond the input.
- Secret keys never go into the front-end. Only publishable keys via `import.meta.env.VITE_*`. Add `.env.example`, never commit `.env`.
- Show clear states: laadimine (loading), õnnestus (success), ebaõnnestus (failed – with "Proovi uuesti").
- Pay button text includes total: "Maksa 32,39 €".
- After success → `/tellimus-kinnitatud` page with order summary and selected locker.

## 11. Routes (Estonian slugs)

| Path | Page |
|---|---|
| `/` | Avaleht (home with carousel) |
| `/toode/:slug` | Product detail |
| `/ostukorv` | Cart |
| `/kassa` | Checkout |
| `/tellimus-kinnitatud` | Order confirmation |
| `/muugitingimused` | Terms of sale |
| `/privaatsus` | Privacy policy |
| `*` | 404 "Lehte ei leitud" |

## 12. Legal & trust content (Estonia / EU)

Include placeholder pages/sections the owner will fill in:
- Prices shown **including VAT** ("Hinnad sisaldavad käibemaksu"). Keep VAT rate as a config constant — verify the current Estonian rate before launch.
- **14-day right of withdrawal** (taganemisõigus) in Müügitingimused.
- Seller company info in footer: company name, registry code, address, e-mail (placeholders).
- Privacy policy + cookie notice (GDPR). No non-essential cookies/trackers before consent.
- Toy safety note on product pages: age recommendation, materials, care instructions ("Pesta käsitsi 30 °C"), CE marking info where applicable.
- Checkbox in checkout: "Olen tutvunud müügitingimustega" (required, unticked by default).

## 13. Accessibility & quality

- Semantic HTML (`header`, `nav`, `main`, `footer`, `button` — never clickable `div`s).
- All interactive elements keyboard accessible with visible focus ring (lilac outline).
- Form inputs have labels and Estonian error messages (e.g. "Palun sisesta kehtiv e-posti aadress").
- Images: `alt` in Estonian, `loading="lazy"`, explicit width/height to avoid layout shift.
- Target Lighthouse ≥ 90 for Performance, Accessibility, Best Practices, SEO.
- SEO: page `<title>` and meta description in Estonian, e.g. "Minu esimene kaisukas – käsitööna kootud mänguasjad".

## 14. Code conventions

- Functional components + hooks only. One component per file, `PascalCase.tsx`; hooks `useSomething.ts`.
- Code identifiers, comments and commit messages in **English**; UI text in **Estonian** (via `et.ts`).
- Strict TypeScript, no `any`. Props typed with `interface`.
- Keep components small (< ~150 lines); extract logic into hooks.
- No `console.log` left in committed code.
- Write tests for: cart reducer, price formatting, products count (= 10), checkout validation, carousel navigation.

## 15. Commands

```bash
npm install
npm run dev       # local development
npm run build     # production build (local only, no deploy)
npm run preview   # preview build locally
npm run test
npm run lint
```

## 16. What Claude must NOT do

- Do not deploy or add deployment configuration.
- Do not show more or fewer than 10 products in the carousel.
- Do not use colors outside the lilac palette/tokens.
- Do not write UI text in English or hard-code strings in components.
- Do not handle raw card data or put secret keys in front-end code.
- Do not add delivery methods other than parcel lockers unless asked.
- Do not add large dependencies or UI frameworks without asking.

## 17. Workflow for Claude

1. Read this file and the relevant existing code before editing.
2. Make small, focused changes; keep the app building (`npm run build`) and tests passing.
3. After changes, summarise: what changed, new files, new dependencies, anything mocked that must be replaced before going live.
