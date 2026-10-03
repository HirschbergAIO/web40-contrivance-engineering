# Goodbye App Stack: Unbundle Shopify to Headless Next.js on Vercel, with an AI Assistant as Your CMS

The average Shopify store isn't really a website anymore — it's a subscription stack. The theme customizer holds the design hostage, and a dozen apps at $10–50 a month each hold the functionality hostage: reviews, upsells, search, popups, email capture, loyalty. Every app injects its own scripts, its own dashboard, its own renewal. The store gets slower and more expensive every quarter, and the owner can't change a homepage banner without a tutorial.

This is the third article in the Web 4.0 Contrivance Engineering series (the first covered WordPress, the second static HTML sites). Same destination — Next.js on GitHub + Vercel, with an AI assistant as the CMS — but the angle here is different: **unbundling**. A store isn't just content; it's a transaction machine. So we split it in two. Content and merchandising move into the chat-and-git loop. Commerce stays on battle-tested rails — headless Shopify via the Storefront API — at least at first. The casualty here isn't checkout. It's the theme customizer and the app stack.

---

## The frame: unbundle the transaction machine

The series thesis, restated for commerce: **contrivance engineering** is the discipline of replacing an entire software category with a smaller, cleverer mechanism. A CMS was only ever two jobs — **store content with structure** (git does it better than anything) and **give non-technical people a way to change it** (language beats dashboards). The loop was always *intent → change → review → publish*; the contrivance rebuilds it from four cheap parts: a chat, a branch, a preview URL, and a merge.

A Shopify store welds two machines together: the **merchandising machine** (homepage, collections, product copy, banners, landing pages — all content) and the **transaction machine** (cart, checkout, payments, fraud analysis, order management — all money). The merchandising machine is a CMS problem wearing a commerce costume, and it surrenders to the contrivance completely. The transaction machine is where real money changes hands, and you don't rebuild that on a weekend — you keep it on Shopify's rails via the Storefront API.

So the honest path has two phases. **Phase 1:** headless Shopify — new Next.js storefront, same checkout, same payments, same fraud analysis, same PCI scope (which is to say: Shopify's, not yours). **Phase 2:** a full replatform later, if ever, once the new storefront has earned its keep. This article is Phase 1. Anyone who tells you to replatform a working checkout in one move is selling you something.

---

## Part 1 — The architecture

The four familiar pieces, plus one guest:

- **Next.js** renders the storefront — product pages, collections, landing pages, blog
- **GitHub** is the database for everything that isn't transactional: page copy, banners, product descriptions, merchandising content, all versioned
- **Vercel** hosts the storefront, with a preview deployment for every change
- **An AI assistant** is the CMS — merchandising happens in conversation
- **Shopify** remains the commerce backend, reached over the Storefront API: catalog, variants, inventory, cart, and checkout

```
Owner  →  (chat)  →  AI assistant  →  edits content in GitHub repo
                                              ↓
                                    Vercel builds a preview URL
                                              ↓
                              Owner reviews → merge → production deploys

Storefront  →  Storefront API  →  Shopify (products, inventory, cart, checkout)
```

**Product data** stays in Shopify — it's the system of record for SKUs, variants, prices, and inventory. The Next.js frontend fetches it at build or request time. **Merchandising copy** — the words around the products — lives in the repo as Markdown/MDX, where the assistant can rewrite forty product descriptions in one sitting.

A minimal Shopify client — `lib/shopify.ts`:

```ts
// Pin the API version to the latest stable release
const SHOP = process.env.SHOPIFY_STORE_DOMAIN!;      // your-store.myshopify.com
const TOKEN = process.env.SHOPIFY_STOREFRONT_TOKEN!;  // storefront access token

export async function shopify(query: string, variables = {}) {
  const res = await fetch(`https://${SHOP}/api/2026-07/graphql.json`, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "X-Shopify-Storefront-Access-Token": TOKEN,
    },
    body: JSON.stringify({ query, variables }),
    next: { revalidate: 60 },
  });
  const { data } = await res.json();
  return data;
}
```

Product pages fetch live data; cart actions use the Storefront API's cart mutations, and checkout hands off to Shopify's own checkout via the returned `checkoutUrl` — the customer never notices the seam, and neither does your PCI scope.

### Why this beats the app stack for growing stores

| | Shopify theme + apps | Headless Next.js + Shopify backend |
|---|---|---|
| Storefront speed | Theme + N app scripts, often 3–6s | <1s, edge-rendered, no app bloat |
| App costs | $10–50/mo per app, stacking forever | Replace or drop most; keep only what earns |
| Merchandising | Theme customizer + app dashboards | A conversation; preview URL per change |
| Design freedom | Whatever the theme allows | Anything — it's your codebase |
| Checkout | Shopify (unchanged) | Shopify (unchanged) |

---

## Part 2 — The migration playbook

### Step 1: Audit the store

1. **Catalog inventory** — product count, variant depth, custom product types. A 40-SKU store and a 4,000-SKU store are different migrations.
2. **Theme customizations** — list every customized section and template. Screenshot the homepage, product page, and collection page; these become your component specs.
3. **App inventory** — for each installed app, write down what it *does*, not what it's called. "Loox" → "photo reviews". "Rebuy" → "post-purchase upsells". "Privy" → "email popups". Count the monthly total — you'll want that number for Part 5.
4. **URL inventory** — Shopify's patterns are predictable: `/products/{handle}`, `/collections/{handle}`, `/pages/{handle}`, `/blogs/{blog}/{article}`. Export them; you'll retain them verbatim in Step 6.
5. **Traffic inventory** — top 20 product and collection pages. Those get rebuilt first and tested hardest.

### Step 2: Export the catalog and content

Shopify admin → **Products → Export** gives you the full catalog as CSV: handles, titles, variants, SKUs, prices, inventory, image URLs. Blog posts and pages export similarly. Hand the CSV to the assistant: *"Convert this catalog into structured product content files — one per product, handle as filename, description converted to Markdown, variants and SKUs in frontmatter."* It writes and runs the conversion script.

**Media:** product images come down with the export URLs. Move them into the repo (or object storage for large catalogs) and let `next/image` serve them.

### Step 3: Rebuild the storefront — theme sections become components

Map the theme to Next.js the way the WordPress article mapped templates to routes: theme sections (hero, featured collection, testimonials) become React components; product and collection templates become dynamic routes (`app/products/[handle]/page.tsx`, `app/collections/[handle]/page.tsx`) fed by the Storefront API.

Same fork as ever: **faithful rebuild** (same design, zero customer disruption — hand the assistant the theme CSS or screenshots for Tailwind equivalents) or **redesign** (cheapest moment you'll ever get, since the frontend is being rebuilt anyway). The pragmatic rule holds: real revenue → faithful first, redesign second.

### Step 4: Apps — keep, replace, or kill

Go through the Step 1 inventory and give every app one of three verdicts:

- **Keep** — apps that are genuinely headless-friendly and earn their fee. Klaviyo (email/SMS) has real APIs and works fine headless. Anything whose value is in *data and deliverability*, not in *injecting storefront widgets*, usually stays.
- **Replace** — apps whose job was storefront widgets. Reviews → Judge.me (headless-capable) or a lightweight custom review block reading Shopify's own review data. Search → Algolia, or a custom search over the catalog you already fetch. Upsells → rebuild as native components at checkout-adjacent touchpoints you now control.
- **Kill** — the popup app, the announcement-bar app, the "trust badge" app. These were $15/mo patches for not owning your frontend. You own it now.

Typical outcome: a 12-app stack becomes 3–4 kept integrations. That's the unbundling, in dollars.

### Step 5: The checkout decision — don't get clever

Keep Shopify's checkout. Full stop, for Phase 1. This is where the money moves, and it comes with PCI compliance, fraud analysis, Shop Pay, and a decade of edge cases you do not want to rediscover. The headless flow is: cart mutations via the Storefront API → `cartCreate` returns a `checkoutUrl` → customer completes purchase on Shopify's checkout → webhooks tell your systems what happened.

Anyone proposing a custom checkout in Phase 1 is proposing risk you can't price. Revisit in Phase 2, if ever.

### Step 6: URLs — retain the handles

Shopify's URL patterns are clean and SEO-valuable. Keep them verbatim: `/products/{handle}` and `/collections/{handle}` become your Next.js dynamic routes with identical paths. The series rule stands — the best redirect is the one you never need. Redirect only what genuinely changes (custom `/pages/` slugs you rename, blog URL restructuring).

### Step 7: Deploy — GitHub + Vercel, branches included

Push to GitHub, import into Vercel, point DNS. Same development → staging → production ladder as the series standard: feature branches get preview URLs, `staging` is the merchandising review environment, `main` is production, branch protection on both. The assistant works on branches; nothing ships without a preview the owner has seen.

One addition for stores: **environment variables per environment** — staging talks to a Shopify development store or a staging context, production talks to the live store. Never let a preview deployment write test orders into production analytics.

### Step 8: Cutover + the pitfalls list

1. **Inventory freeze** — agree a quiet window; no catalog edits during final sync.
2. **Final catalog export** → rebuild → verify on staging, especially the top-20 pages and the full checkout path with a real test order.
3. **Switch DNS**, submit sitemaps, watch 404s for two weeks.

Pitfalls that sink store migrations:

- **Inventory sync during the freeze.** If orders keep flowing on the old storefront mid-migration, reconcile before cutover — overselling during launch week is the nightmare scenario.
- **Variant SKU mismatches.** The CSV export is truth; if handles or SKUs drifted between apps, the new frontend shows the wrong variant. Reconcile SKUs in Step 2, not Step 8.
- **App webhooks still firing.** Uninstalled-but-not-really apps (looking at you, email popups) can keep capturing emails or firing discounts from the old theme. Audit webhooks in the Shopify admin post-cutover.
- **Discount codes and gift cards.** They live in Shopify and keep working — but *test* them on the headless checkout path. Test them twice.
- **Email capture forms.** Every Klaviyo/Privy form gets rebuilt and re-tested; a silent signup failure costs you the list growth you can't see.
- **Email, as always.** MX records before DNS changes. Every time.

---

## Part 3 — The AI assistant as your merchandiser

This is where the contrivance pays for itself monthly. Merchandising used to mean logging into three dashboards. Now it's a conversation:

> **Owner:** "Fall drop launches Friday. Write collection copy for the six new products, rewrite their descriptions in our voice — keep every spec and measurement intact — and draft the homepage banner."
>
> **Assistant:** drafts everything on a branch, commits, Vercel builds a preview of the actual storefront with the new copy in place.
>
> **Owner:** opens the preview, replies "banner's too shouty, soften it."
>
> **Assistant:** revises, preview rebuilds in ~30 seconds.
>
> **Owner:** "Ship it." → staging review → merge → production.

Bulk merchandising — the thing that used to take a VA a week — is now a paragraph.

### The merchandising prompt library — steal these

- *"Rewrite these 40 product descriptions in our voice. Keep every spec, measurement, and material intact; only the selling copy changes. Open a PR against staging."*
- *"Draft collection page copy for the fall drop: 120 words, our tone, mention the new colorways. Here's the product list: [paste]."*
- *"Flag every product with missing alt text or missing meta descriptions, and draft both."*
- *"Write a landing page for the holiday sale — hero, three value props, FAQ. Match the tone of last year's page."*
- *"Audit pricing display consistency: find products where the compare-at price logic looks wrong."*
- *"Turn our five best reviews this month into homepage testimonial copy."*
- *"Generate SEO titles and descriptions for all products in the clearance collection."*
- *"Draft the abandoned-browse email sequence outline for the new collection — three emails, our voice."*
- *"Check every collection page for products that are out of stock and shouldn't be featured."*

---

## Part 4 — Honest tradeoffs: when headless isn't worth it

- **Don't replatform a high-volume checkout lightly.** If the store does serious revenue, Phase 1 keeps Shopify checkout for a reason. The day you touch checkout is the day you hire specialists, not the day you follow a blog post.
- **App ecosystem depth is real.** Some apps do genuinely hard things (subscriptions, B2B pricing, complex bundles). If three load-bearing apps have no headless path, price that rebuild honestly before committing.
- **Tiny catalog, happy owner.** A 12-product store whose owner loves the Shopify admin and changes copy twice a year? The ROI clock runs on merchandising velocity. Be honest about it.
- **Shopify's own answer exists.** Hydrogen (Shopify's Remix-based headless framework) is the official path. This guide picks Next.js + Vercel for the wider ecosystem, the edge network, and the AI-CMS workflow the series is built on — but Hydrogen is a legitimate choice, not a strawman.
- **Someone still approves merges.** Same as every article in the series: if nobody clicks "approve," nothing ships.

---

## Part 5 — Pitching it: your store is a subscription stack

Store owners feel the app-stack pain as a line item. Your pitch is arithmetic plus relief:

- **"Add up your apps."** Walk through the Step 1 inventory together. Twelve apps at $10–50 a month is $2,000–7,000 a year for widgets — before the theme, before the plan. The unbundling usually kills half of it in week one.
- **"You'll merchandise at the speed of conversation."** New drop Friday? It's a paragraph, a preview link, and an approval — not a VA, three dashboards, and a week.
- **"Checkout doesn't change."** This is the sentence that closes the deal. Same Shopify checkout, same Shop Pay, same fraud protection. All the risk they imagined evaporates.
- **"You own the storefront."** It's in *their* GitHub repo. No theme lock-in, no app lock-in for presentation, any developer can take over.

The objections are the series standards: *"What if the AI gets it wrong?"* — nothing publishes without approval; safer than the theme customizer's Publish button. *"What if you disappear?"* — their repo, their content, Shopify still Shopify. And the cost story writes itself:

| | Theme + app stack, per year | The unbundling |
|---|---|---|
| Apps | $1,200–7,200 in subscriptions | Keep 3–4 that earn; kill the rest |
| Theme | $180–350 + customization fees | $0 — it's your codebase |
| Merchandising labor | VA hours or agency retainers | Conversation-speed, in-house |
| Checkout risk | — | Unchanged (still Shopify) |

---

## Getting started this weekend

1. `npx create-next-app@latest` — scaffold it, and create a Shopify headless channel for your Storefront API token.
2. Export one product via Products → Export, render it at `/products/[handle]`.
3. Push to GitHub, import into Vercel, open the live URL next to the real store.
4. Then hand your AI assistant the catalog CSV and say: *"build the rest of the storefront."*

The future CMS isn't a dashboard. It's a conversation — with git as the memory and the edge as the server.

---

*Third in the Web 4.0 Contrivance Engineering series. Built the way it describes: written with an AI assistant, versioned on GitHub, deployed on Vercel.*
