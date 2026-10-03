# Goodbye Template Cage: Move Squarespace Sites to Next.js + GitHub + Vercel, with an AI Assistant as Your CMS

Nobody chooses Squarespace by accident. Photographers, restaurants, studios, designers — they chose it for one reason: beauty. A Squarespace site looks good on day one with zero effort, and that is a genuine achievement.

But the template is a cage. What you see is all you get: the layout you want doesn't exist, the feature you need isn't on the roadmap, the plan tier keeps creeping up, and your entire business lives on rented land. This is the fifth article in the Web 4.0 Contrivance Engineering series. The pitch of this one, in a sentence: **keep the beauty, lose the cage.**

---

## The frame: beauty without rent

The series thesis, compressed. **"Contrivance engineering"** = replacing an entire software category with a smaller, cleverer mechanism. A CMS was only ever two jobs:

1. **Store content with structure.** Git does it better than any database: every version, every author, every change, diffable forever.
2. **Give non-technical people a way to change it.** The dashboard was the answer in 2005. The better answer now is language itself — you describe the change, the assistant makes it, you review the result.

Content management was never really about the dashboard. It was always about the loop: *intent → change → review → publish.* The contrivance rebuilds it from four cheap parts: a chat, a branch, a preview URL, and a merge.

For Squarespace, the frame lands differently. Squarespace didn't sell you a database — it sold you *taste*. The cage was never the content model; it's the rigidity: the section that can't do what you want, the gallery layout that almost fits, the monthly fee for the privilege. The contrivance keeps the part you paid for — the beauty, rebuilt faithfully — and discards the part you didn't: the rent, the rigidity, the roadmap you don't control.

---

## Part 1 — The architecture

Same four pieces as the rest of the series:

- **Next.js** renders the site (static where possible, dynamic where needed)
- **GitHub** is the database — content lives in Markdown files, versioned like code
- **Vercel** hosts it, with a preview deployment for every change
- **An AI assistant** is the CMS — content changes happen through plain conversation

What's different here — and it matters more than in any other article in this series: **design fidelity is the whole job.** A Squarespace client chose beauty. If the rebuilt site is 95% of the original, you've failed — the 5% is what they were paying for. So the migration starts differently: before anything moves, the AI assistant studies the live site and extracts the design system — type scale, spacing rhythm, color palette, button treatments, gallery behaviors — into a documented spec. That spec becomes the acceptance test for every rebuilt component.

```
You  →  (chat)  →  AI assistant  →  edits Markdown/MDX + components in GitHub repo
                                              ↓
                                    Vercel builds a preview URL
                                              ↓
                              You review → merge → production deploys
```

**Content** lives in the repo as Markdown (or MDX) files with frontmatter — same as the other articles. Galleries become structured data (image + caption + alt text) rendered by a gallery component. That separation is the quiet upgrade: on Squarespace, a gallery's *content* was fused to its *presentation*, which is why every redesign meant rebuilding every gallery by hand. Here, you restyle once and every gallery follows.

---

## Part 2 — The migration playbook

### Step 1: Audit — find the crown jewels

1. **Page inventory** — every page and its section layout. Note which pages actually earn their keep.
2. **Galleries and portfolios** — the crown jewels for this audience. Inventory every gallery: image count, captions, categories, click-through behavior.
3. **Blog** — if they blog, note post count, categories, tags.
4. **Commerce / Scheduling** — Squarespace Commerce or Scheduling (Acuity) present? These are the hard edges (see Step 5).
5. **Forms** — contact, booking inquiry, newsletter signup. Note where submissions go today.
6. **URL inventory** — Squarespace blog collections typically live at `/blog/{slug}`; pages at `/{slug}`. Record them all; you'll retain them verbatim in Step 6.

### Step 2: Export — use it, then repair it

Squarespace has an export: **Settings → Advanced → Import/Export → Export**, which produces a WordPress-format (WXR) XML file. Use it — but know what it is: **lossy**. It carries pages, blog posts, and images reasonably well. It does *not* cleanly carry gallery layouts, section styling, summary blocks, or anything that made the site beautiful. Say it plainly to clients: the export rescues the *words and pictures*, not the *design*.

Hand the XML to the AI assistant: *"convert every post and page to Markdown with title, date, slug, and tags in frontmatter; download and relink every image; flag anything that didn't survive."* It writes the conversion script, runs it, and produces the damage report. The damage report is the real deliverable — it's your rebuild checklist.

**Media:** Squarespace serves scaled images; get the originals. Download full-resolution originals from the media library **before** you cancel anything (see Step 8).

### Step 3: Rebuild the design faithfully — spend the care budget here

This is where the hours go, and it's honest work: template → Next.js components. The design-system spec from Part 1 drives it — the assistant generates Tailwind styles matching the extracted type scale, palette, and spacing, and you verify against the live site side by side.

Rebuild order: layout shell (header/footer/nav) → homepage hero → gallery components → inner pages → blog templates. Every component gets reviewed on a Vercel preview URL against the original. Pixel-careful isn't a slogan here; it's the acceptance criterion.

The redesign question applies, same as the WordPress article: faithful first is the safe default for established businesses. A redesign is a separate project with its own approval cycle — don't smuggle it into the migration.

### Step 4: Galleries and portfolios — the crown jewels, upgraded

Galleries become data + component: each image with caption and alt text in frontmatter, rendered by a gallery component using `next/image` (proper sizing, WebP/AVIF, blur placeholders). Two upgrades fall out for free:

- **Real alt text.** Have the assistant write descriptive alt text for every portfolio image. For photographers, this is genuinely valuable SEO — image search is a traffic source, and Squarespace galleries were alt-text deserts.
- **Performance.** Properly sized responsive images with blur-up placeholders load visibly faster than the template's one-size-fits-all output.

### Step 5: Backends — forms move, commerce gets a decision

- **Squarespace Forms** → a React form + Next.js Server Action, sending via Resend or Formspree. Same pattern as the other articles; test every form on staging.
- **Scheduling (Acuity)** → keep it. Embed the Acuity scheduler on the new site exactly as before. Don't migrate what isn't broken.
- **Commerce** → be honest: this is a project, not a step. Options: keep checkout on Squarespace (subdomain or embedded), or replatform to Shopify/Stripe with a headless storefront. Don't pitch a commerce migration as a weekend job — scope it separately.

### Step 6: URLs — retain verbatim

Squarespace URLs are already clean: `/blog/{slug}` for posts, `/{slug}` for pages. Keep them exactly — the series rule stands: the best redirect is the one you never need. Map anything that does change in `next.config.ts` redirects, and have the assistant generate the array from your inventory CSV:

```ts
const nextConfig = {
  async redirects() {
    return [
      { source: "/old-page", destination: "/new-page", permanent: true },
    ];
  },
};
```

### Step 7: Deploy — GitHub + Vercel, branches included

Push to GitHub, import into Vercel (it auto-detects Next.js), point DNS. Set up the same development → staging → production branch ladder as the rest of the series:

| Branch | Environment | Deploys to |
|---|---|---|
| `main` | **Production** | Your domain, via Vercel production deployment |
| `staging` | **Staging** | A staging URL for visual review against the original |
| Feature branches | **Development** | Automatic Vercel preview URLs — one per pull request |

Protect `main` and `staging` (GitHub → Settings → Branches → branch protection rule): require a pull request, require an approval, require the Vercel build to pass. For this audience, staging is where the client reviews the *look* — side by side with the original — before anything ships.

### Step 8: Cutover + the pitfalls list

1. **Content freeze** on Squarespace; final export; merge to `staging`.
2. **Client reviews the full staging site against the original** — page by page. This is the sign-off gate.
3. **Merge `staging` → `main`**; switch DNS (copy *all* records first — Squarespace often hosts the domain and its DNS; MX records for Google Workspace must survive).
4. **Search Console:** submit the new sitemap, request indexing on key pages.
5. **Watch 404s for two weeks** and add redirects for anything missed.
6. **Then** cancel the Squarespace subscription — after DNS has moved, after originals are downloaded, after 30 days of parallel calm.

**Pitfalls:**

- **Image originals.** Download full-res *before* canceling. After cancellation, they're gone.
- **Font licensing.** Squarespace bundles Google Fonts (open, fine) and sometimes Adobe Fonts — check the license before self-hosting.
- **Summary blocks and carousels** don't export; they're rebuild items, not migration items. Budget them.
- **The domain.** If Squarespace is the registrar, transfer it or keep DNS there — decide deliberately, not by accident.
- **Email, as always.** MX records. Every time.

---

## Part 3 — The AI assistant as your CMS

The daily workflow for a visual business:

> **Owner:** "Swap the homepage hero gallery for the fall collection shots."
>
> **Assistant:** updates the gallery data file on a branch, optimizes the new images, writes alt text, opens a preview.
>
> **Owner:** checks the preview. "The third one — swap it for the barn shot."
>
> **Assistant:** revises; preview rebuilds in ~30 seconds.
>
> **You:** merge → staging → production.

No template editor. No "that layout isn't available on your plan."

### The Squarespace prompt library — steal these

- *"Swap the hero gallery for these new images [attached]. Optimize, write alt text, open a PR against staging."*
- *"Update the menu page with the new seasonal items: [paste]. Keep the formatting consistent."*
- *"Add this testimonial to the homepage: [quote, name]."*
- *"Change our hours everywhere they appear to [new hours]."*
- *"Audit every gallery for missing alt text and write it — descriptive, photographer-grade."*
- *"Find every image over 400KB site-wide, convert to WebP, report bytes saved."*
- *"Draft a blog post announcing [event] in our voice, with a meta description under 160 chars."*
- *"Check all external links; flag the dead ones."*
- *"Regenerate the sitemap and confirm every URL returns 200 on staging."*

---

## Part 4 — Honest tradeoffs

This stack isn't for everyone. Be straight with clients about the limits:

- **Squarespace is genuinely good at "beautiful with zero effort."** The AI-chat workflow replaces *content edits* beautifully. It does not replace the specific joy of a non-technical person dragging a gallery block into place at midnight. If the client lives for that, say so.
- **Don't undersell the design labor.** A faithful rebuild is real work — design-system extraction, component building, side-by-side review. Quote it honestly; the cage-free ongoing story is where the value compounds.
- **Commerce and Scheduling are edge cases.** Acuity embeds fine. Real commerce is a separate scoping conversation (see Step 5).
- **Someone still approves merges.** The preview-approval loop needs a human who clicks. Set that expectation on day one.

---

## Part 5 — Pitching it: keep the beauty, lose the cage

- **"Same look. None of the limits."** The site they love, rebuilt faithfully — and now the layout they always wanted is a conversation away instead of a template limitation.
- **"Stop paying rent on your own website."** The plan tier, the feature paywalls, the renewal — gone. Hosting drops to Vercel's free tier; the site becomes an asset they own in their GitHub repo.
- **"Your content finally belongs to you."** Markdown files, full history, downloadable everything. No export anxiety ever again.

**Handling the two objections you'll hear:**

**"What if the AI gets it wrong?"** Nothing publishes without their preview approval — safer than Squarespace, where anyone with contributor access edits the live site directly.

**"What if you disappear?"** Everything is in their GitHub account — code, content, full history. The content is Markdown, the hosting is Vercel, the "CMS" is a documented conversation workflow. No license keys, no proprietary lock-in. Any developer can take over.

---

## Getting started this weekend

1. `npx create-next-app@latest` — scaffold it.
2. Run the Squarespace export (Settings → Advanced → Import/Export); hand the XML to the assistant: *"convert it and report the damage."*
3. Have the assistant extract the design system from the live site; rebuild the homepage hero.
4. Push to GitHub, import into Vercel, compare the preview against the original side by side.

The future CMS isn't a dashboard. It's a conversation — with git as the memory and the edge as the server.

---

*Fifth in the Web 4.0 Contrivance Engineering series. Built the way it describes: written with an AI assistant, versioned on GitHub, deployed on Vercel.*
