 # Pastel & Co. Lifestyle Store

Pastel & Co. is a full-stack, responsive lifestyle storefront for thoughtful everyday objects. The experience uses a soft cream canvas, signature pastel mint actions, warm rose accents, editorial serif headings, product-led imagery, and a calm, tactile shopping flow.

## Included experience

The storefront includes the home page, shop/catalog, product detail, cart, three-step checkout, order success, customer profile, wishlist, and studio admin overview routes:

- `/` — offset hero, curated categories, best sellers, trust badges, newsletter, and editorial brand story
- `/shop` — search, category/color/price/in-stock filters, sorting, product grid, and quick-view modal
- `/product/:id` — gallery, variants, quantity, cart/buy actions, accordions, ratings, reviews, and review form
- `/cart` — localStorage-persisted cart, quantity controls, promo code `SOFT10`, shipping estimate, and totals
- `/checkout` — Shipping Info → Payment Details → Order Confirmation wizard with client and server validation
- `/order-success` — receipt, tracking code, delivery timeline, and order follow-up
- `/profile` — seeded order history, statuses, address, and wishlist
- `/admin` — seeded sales metrics, revenue chart, recent orders, and inventory table

## Tech stack

- React 19 + Vite + TypeScript
- Wouter client routing
- Express runtime with the initialized tRPC/auth platform helpers preserved
- Drizzle ORM for MySQL-compatible managed database tables
- Tailwind v4 primitives plus a custom CSS design system
- Lucide icons and Sonner toasts

## Local setup

The initialized project uses pnpm 10.18.0.

```bash
pnpm install
pnpm dev
```

The development server listens on `PORT` when supplied and defaults to `3000`.

Useful commands:

```bash
pnpm check       # TypeScript diagnostics
pnpm test        # Vitest suite
pnpm build       # Vite frontend + Express production bundle
pnpm db:migrate  # Apply checked-in Drizzle migrations
pnpm db:push     # Generate/apply schema changes intentionally
```

## Environment variables

The managed Webdev runtime supplies the platform environment for the selected server and database resources. Local development can use the generated platform setup, or a local MySQL-compatible `DATABASE_URL` when running Drizzle migrations. Never commit secrets.

- `PORT` — HTTP listener port; defaults to `3000`
- `DATABASE_URL` — MySQL-compatible database DSN for Drizzle migrations and database-backed extensions
- Platform OAuth/service values — supplied by the managed runtime; do not place them in browser code

## API surface

The Express runtime provides:

- `GET /api/health` — unauthenticated readiness response
- `GET /api/products` — filtering via `q`, `category`, `color`, `maxPrice`, `inStock`, sorting via `sort`, and pagination via `page`/`pageSize`
- `POST /api/cart` — validates product IDs, quantities, and stock availability
- `POST /api/checkout` — validates shipping fields, mock payment details, cart items, and returns an order ID
- `/api/trpc` — preserves the initialized platform tRPC router and OAuth-aware auth procedures

## Mock checkout

No live purchase is performed. The checkout accepts a Stripe-like test shape:

- Success: `4242 4242 4242 4242`, any future `MM/YY`, and a 3–4 digit CVC
- Deterministic failure: use a card ending in `0000`
- The UI displays a clear “mock payment” message and the server repeats validation before returning success.

## Database schema

`drizzle/schema.ts` preserves the generated `users` table and adds:

- `products` — title, description, price, category, stock, image JSON, color family
- `orders` — user/email, order item snapshot JSON, total, status, created timestamp
- `reviews` — product reviews and ratings
- `addresses` — saved shipping addresses
- `wishlistItems` — persistent wishlist records
- `cartSessions` — durable cart/session records for future authenticated persistence

The checked-in `drizzle/0001_commerce.sql` creates the commerce tables. The current Preview UI uses a curated catalog in `client/src/data/products.ts` so public shopping remains useful even before seed records are added to a local database.

## Brand and asset notes

- Signature brand color: `#A8E6CF` pastel mint
- Secondary accent: `#FFAAA5` soft rose
- Canvas: `#FAF9F6` cream
- Text: `#2D3748` deep charcoal
- Project route manifest: `client/public/manus-routes.json`
- Project logo metadata: `app.config.ts`
- Homepage hero artwork uses a public image URL so it also renders in Netlify static hosting.

## Netlify deployment

The repository includes `netlify.toml` for a static frontend deployment. In Netlify, set the **Base directory** to the folder containing this `package.json` (the project root), then use the checked-in settings: build command `pnpm build:static` and publish directory `dist/public`. Do not use `pnpm --filter pastelshop... run build`; this is a single-package project, not a multi-package workspace filter target. If the project is nested inside another repository, either move its contents to the repository root or set Netlify Base directory to `pastelshop`.

The Netlify configuration publishes the Vite storefront and supports client-side routes through the SPA fallback. It does not host the Express APIs, Manus OAuth callback, database, or server-enforced admin RBAC. Use the Manus Preview/managed server deployment or another Node-capable host for the full-stack behavior; Netlify is suitable for the static storefront shell unless the backend is separately deployed and proxied.

## Production notes

The managed Webdev Dockerfile remains the full-stack production entrypoint. `pnpm build` creates the Vite output and bundles the Express runtime into `dist/index.js`; the runtime honors `PORT` and serves `/api/health` without authentication. Preview remains the verified handoff unless a separate publish operation succeeds. Auto-publish is not enabled by this implementation.

## Authentication and admin access

Authentication uses the initialized Manus OAuth flow rather than a separate password system. The existing application session cookie remains `webdev_app_session`, is validated with the managed `MANUS_JWT_SECRET`, and is configured for cross-site Preview embedding with `SameSite=None; Secure`. The header exposes Sign in/Sign out state through the starter `useAuth` hook, while `/profile` displays the current authenticated account.

The admin dashboard has two layers of protection:

1. The client route at `/admin` checks the current Manus session and shows a sign-in prompt when logged out.
2. The dashboard data comes from the `admin.dashboard` tRPC procedure, which uses the starter `adminProcedure` middleware and rejects both anonymous users and authenticated users whose persisted `users.role` is not `admin`.

Role assignment is application data. The existing user sync preserves an existing admin role and automatically assigns `admin` to the configured `OWNER_OPEN_ID` when that owner signs in for the first time. Other accounts default to `user`. Set `OWNER_OPEN_ID` only through the protected managed environment when you need to designate the project owner as an admin; never expose it to the browser.
