# Goodbye Lock-in: Escape Wix to Next.js + GitHub + Vercel, with an AI Assistant as Your CMS

Wix's pitch was always "a website in an afternoon." What the homepage never stated was the price: **you can never leave.** There is no export button. Your words live in Wix's database, your design lives in Wix's editor, and the day you decide to go, you discover the door was painted on. Try to take your site with you and Wix shrugs: rebuild it elsewhere, by hand, from scratch.

This is the fourth article in the Web 4.0 Contrivance Engineering series (Article 1 covered WordPress, Article 2 covered static HTML sites). Same destination — Next.js on GitHub + Vercel, with an AI assistant as the CMS — but a different operation entirely. This isn't a migration. It's an **escape**. And the AI assistant is the getaway driver: it crawls your public site and liberates everything Wix wouldn't export into Markdown files you own.

---

## The frame: jailbreaking the CMS

The series thesis, restated for the locked-in: **contrivance engineering** is the discipline of replacing an entire software category with a smaller, cleverer mechanism. A CMS was only ever two jobs — **store content with structure** (git does it better than anything) and **give non-technical people a way to change it** (language beats dashboards). The loop was always *intent → change → review → publish*, and the contrivance rebuilds it from four cheap parts: a chat, a branch, a preview URL, and a merge.

Wix fused those two jobs into a prison: storage you can't access, plus an editor you can't leave. The contrivance doesn't *replace* Wix's CMS — it **jailbreaks** it. Everything you built is visible on the public web. The assistant reads what Wix wouldn't export and writes it into files that live in your GitHub repo. Same four cheap parts on the other side of the wall.

---

## Part 1 — The architecture

Same four pieces as the rest of the series:

- **Next.js** renders the site (static where possible, dynamic where needed)
- **GitHub** is the database — content lives in Markdown files, versioned like code
- **Vercel** hosts it, with a preview deployment for every change
- **An AI assistant** is the CMS — content changes happen in plain conversation

Two things are different for Wix escapes:

1. **There is no export file and no server access.** No XML dump, no database, no FTP. The public website *is* the source of truth — the assistant crawls it page by page and reconstructs your content from what visitors see. If it's on the public site, it can be liberated. If it's behind a Wix login (members areas, app dashboards), it gets handled manually.
2. **The rent ends.** Wix owners pay a premium plan every month, forever, for the privilege of hosting their own words. The new stack costs $0 on Vercel's hobby tier. The escape isn't just technical — it's financial.

```
Wix (locked box, monthly rent)
        ↓  crawl the public site
Markdown files in YOUR GitHub repo
        ↓
Vercel builds a preview URL → you review → merge → production
```

### Why this beats Wix for content sites

| | Wix | Next.js + GitHub + Vercel + AI |
|---|---|---|
| Cost | Premium plan rent, forever (~$200–400/yr) | $0 (Vercel hobby tier) |
| Ownership | No export — your site is a hostage | Your repo, your Markdown, take it anywhere |
| Speed | Often sluggish, heavy editor runtime | <1s, global edge CDN |
| Content history | Whatever Wix's site history shows you | Full git history, branches, diffs |
| Editing UX | Drag-and-drop wrestling match | A conversation |
| Leaving | Rebuild by hand | `git clone` |

---

## Part 2 — The escape playbook

### Step 1: Audit the hostage situation

Before the crawl, know what you're liberating:

1. **Sitemap inventory** — fetch `/sitemap.xml`. It's the closest thing Wix gives you to an export manifest: every public URL, ready to crawl.
2. **Page types** — static pages, Wix Blog posts (and categories), forms, booking/scheduling pages, members-only areas (these can't be crawled — plan manual rebuilds).
3. **Wix Stores?** — stop here and read carefully. Migrating a store is a replatforming project, not a migration step: product catalogs, checkout, payments, and order history don't crawl cleanly. Your honest options are keeping the store on a Wix subdomain temporarily while the content site escapes, or moving commerce to Shopify (headless or hosted). Don't pitch either as a weekend job — see the tradeoffs section.
4. **Media inventory** — images, videos, PDFs sitting in the Wix media manager. Download originals where you can; note what's CDN-only.
5. **Traffic inventory** — your top 20 pages in analytics. Those get liberated first and tested hardest.

### A design decision first: rebuild it, or redesign it?

Wix designs are template-based — the site was never truly *theirs* in the way a custom theme is. Many Wix owners secretly want a redesign and have just never had a clean moment for one. This is that moment. Same fork as the rest of the series:

**Option A — faithful rebuild.** Recreate the look with Next.js components and Tailwind. Fastest escape, zero brand disruption. Hand the assistant screenshots of each page type and have it generate the components — mechanical work it's fast at.

**Option B — redesign during the escape.** The content is being liberated anyway; this is the cheapest redesign you'll ever get. Go content-first, keep URL structure in mind for Step 6, and let every iteration ride the preview-URL workflow.

**The pragmatic rule:** template fatigue is real — if the owner winces at their own homepage, redesign. If the brand is working, rebuild faithfully and redesign later on the new stack, where every change gets a preview link.

### Step 2: Liberate — crawl the public site

There is no API for this and no credentials to request, because there is nothing to authenticate *to*. The public site is the API. The workflow:

1. Feed the assistant the sitemap: *"Crawl every URL in this sitemap. For each page, extract the copy and image URLs into a Markdown file with title, slug, and meta description in frontmatter."*
2. The assistant writes and runs the crawl script, converts HTML to Markdown, downloads the images, and opens a pull request with the whole liberated content tree.
3. **Wix Blog shortcut:** the blog exposes an RSS feed (usually `/blog-feed.xml`) with clean post content — use it for posts instead of scraping rendered pages. It's the one clean exit Wix accidentally left open.

One honest note on editors: whether the site was built with ADI, the Classic Editor, or Wix Studio makes no difference to the crawl — the public HTML is the public HTML. (ADI sites do tend to be rigid templates, which makes them easier to rebuild — and their owners more open to redesign.)

### Step 3: Scaffold the Next.js app and content layer

```bash
npx create-next-app@latest my-site --typescript --tailwind --app
cd my-site
npm install gray-matter remark remark-html
```

Same content layer as Article 1: Markdown files with frontmatter under `content/`, read with gray-matter, rendered through remark, pre-rendered at build time with `generateStaticParams()`. If you followed the flagship article, this step is copy-paste. Your liberated content lands here:

```
my-site/
├── app/                  # routes rebuilt from the sitemap inventory
├── content/
│   ├── posts/            # liberated Wix Blog posts (via RSS)
│   └── pages/            # liberated static pages (via crawl)
├── public/images/        # liberated media
└── lib/posts.ts          # content helpers (see Article 1)
```

### Step 4: Assets — what you can and can't take with you

- **Images: grab the largest the CDN will serve.** Wix serves images from its CDN at the resolution the page requested — you cannot download the originals. Pitfall: the crawl must request the *largest available* variant (the CDN URL parameters control width — max them out), because this is the best copy you'll ever get. Then convert to WebP/AVIF and serve via `next/image`.
- **Fonts: re-source them.** Many Wix fonts are licensed for use *inside Wix only*. Do not ship Wix's font files — match them with Google Fonts equivalents or buy proper licenses.
- **Videos and documents:** pull these from the media manager while you still have Wix access. Once the premium plan is cancelled, that door closes too.

### Step 5: Replace the backends

- **Wix Forms** → a React form + Next.js Server Action, sending via Resend or Formspree. Submissions land in an inbox instead of the Wix dashboard.
- **Wix Automations** → replace with a real email tool, or drop them. Most are "thanks for subscribing" noise nobody will miss.
- **Wix Blog** → the posts collection from Step 2's RSS liberation.
- **Wix Chat, Bookings, and other apps** → audit each one. Some map cleanly to services; some have no equivalent — see the tradeoffs section before promising anything.

### Step 6: URLs — the one article where cleanup is a feature

The series rule says retain original URLs — the best redirect is the one you never need. Wix is the exception that proves it: blog URLs like `/post/my-post` or legacy `/single-post/...` are genuinely ugly, and cleaning them to `/blog/my-post` is worth doing. This is the ONE series article where URL cleanup is a feature, not a risk.

But the second half of the rule still holds: **redirect every old URL, permanently.** In `next.config.ts`:

```ts
const nextConfig = {
  async redirects() {
    return [
      // Wix blog cruft → clean slugs
      { source: "/post/:slug", destination: "/blog/:slug", permanent: true },
      { source: "/single-post/:slug", destination: "/blog/:slug", permanent: true },
    ];
  },
};
```

Generate the full map from the sitemap inventory — *"here's every old URL, generate the redirects array"* is a one-sentence job for the assistant.

### Step 7: Deploy — GitHub + Vercel, branches included

Push to GitHub, import into Vercel (it auto-detects Next.js), point DNS. Set up the same development → staging → production branch ladder as the rest of the series: feature branches get preview URLs, `staging` is the review environment, `main` is production, branch protection rules on both. The assistant works on branches; nothing ships without a preview you've seen.

### Step 8: Cutover — and actually cancel Wix

1. **Content freeze** — stop editing the Wix site. Do the final crawl for anything that changed.
2. **Verify everything on staging**, especially the top-20 traffic pages and every form.
3. **Merge `staging` → `main`**, switch DNS to Vercel (MX/TXT records preserved — same pitfall as every article in this series).
4. **Search Console:** submit the new sitemap, request indexing on key pages. Watch 404s for two weeks.
5. **Cancel the Wix premium plan.** This is the step people forget: Wix keeps billing until you cancel. Keep one month of overlap as rollback insurance, then cancel and confirm the charge stops.

### Don't skip: the pitfalls list

- **Image resolution ceiling.** The CDN only serves what it serves — max out the width parameters during the crawl. You'll never get a better copy.
- **Font licensing.** Wix-licensed fonts stay with Wix. Re-source everything.
- **JS widgets that can't be exported.** Wix apps render inside Wix's runtime — their *functionality* stays behind even when their *content* is liberated. Each one needs a rebuild-or-replace decision.
- **Members-only pages can't be crawled.** Anything behind a Wix login gets rebuilt manually from what the owner can see.
- **Email, again.** MX records. Every time.
- **The subscription.** Wix bills until *you* cancel it. Put the cancellation on the cutover checklist, not in your memory.

---

## Part 3 — The AI assistant as your CMS

Wix owners don't miss a dashboard — they *escape* an editor. The wrestling match with snap-to-grid, the mystery meat navigation of settings panels, the "why did my text box move" — all of it gets replaced by a conversation:

> **Owner:** "Put the spring sale banner on every page until the end of the month."
>
> **Assistant:** edits the shared banner component on a branch — one change, every page — Vercel builds a preview.
>
> **Owner:** checks the preview on their phone. "Make it red instead of blue."
>
> **Assistant:** revises; preview rebuilds in ~30 seconds.
>
> **Owner:** "Ship it." → merged → staging → production.

No editor. No dragging. No rent.

### The Wix-escape prompt library — steal these

- *"Update the seasonal banner on every page to [text]. Open a PR against staging."*
- *"Rewrite the services page in a warmer tone. Keep every fact, price, and phone number identical."*
- *"Find every page still mentioning the old phone number and update it."*
- *"Draft a 600-word post announcing [X], with a sub-160-character meta description. Open a PR against staging."*
- *"Audit every file under /content for missing meta descriptions and draft them in our voice."*
- *"Our holiday hours changed — update everywhere hours appear."*
- *"Check all external links across the site. For each dead one, suggest a replacement or flag it."*
- *"Regenerate the sitemap from current content and confirm every URL returns 200 on the staging deployment."*
- *"Turn this services page into a FAQ section: [slug]."*
- *"Summarize our three most-visited pages into a new 'Start here' page."*

### Guardrails (because "AI, just publish" needs brakes)

Same four as the rest of the series: branch + preview for everything (nothing reaches `staging` without a preview you've seen, nothing reaches `main` without passing through `staging`); pull-request review where the diff *is* the editorial review; humans stay on facts and voice; everything is `git revert`-able.

---

## Part 4 — Honest tradeoffs: when Wix still wins

- **Wix is genuinely easy for true beginners.** Credit where due: a non-technical person can have a decent site by dinner. The AI-chat workflow replaces *content edits* beautifully, but not the *"I want to drag this box 20 pixels left"* impulse. If the owner's joy is fiddling with layout, they'll miss the editor — be honest about that.
- **Wix Stores is a project, not a migration step.** Catalogs, checkout, payments, order history, and tax rules don't crawl. Treat commerce as a separate replatforming engagement with its own plan and price.
- **Some Wix apps have no equivalent.** Bookings, Restaurants, Hotels by Wix — deep, specific functionality with no one-line Next.js replacement. That's application development, not an escape.
- **Someone has to approve merges.** If the owner won't click "approve" on a preview link, the AI CMS stalls exactly the way any CMS stalls without an editor.

---

## Part 5 — Pitching it: you're renting your website

Every Wix owner who's tried to leave knows the hostage feeling. That's your pitch — you barely have to sell it:

- **"You're renting your website — and you can't take it with you."** Say it plainly. They've felt it. The escape ends the rent *and* the lock-in in one move.
- **"After this, you own the files."** The whole site lives in *their* GitHub repo as Markdown and React. Any developer on earth can take over. No export button needed, ever again, because there's nothing left to export *from*.
- **"Edits without the wrestling match."** No more fighting snap-to-grid at midnight. They describe the change, see a preview, approve — live in minutes.
- **The math sells itself:**

| | Wix, per year | The contrivance |
|---|---|---|
| Platform rent | ~$200–400 (premium plan) | $0 (Vercel hobby) |
| Maintenance | DIY editor wrestling | $0 of busywork |
| Leaving | Rebuild by hand | `git clone` |

### Handling the two objections you'll hear

**"What if the AI gets it wrong?"** Nothing publishes without approval. The assistant drafts on branches, the owner reviews a preview link, merges go through staging. It's *more* controlled than Wix, where anyone with editor access can publish to the live site in one click.

**"What if you disappear?"** Everything is in their GitHub account — code, content, full history. The content is Markdown, the hosting is Vercel, the "CMS" is a documented conversation workflow. No license keys, no proprietary lock-in, no single point of failure with your name on it. For someone escaping lock-in, this lands harder than any feature list.

---

## Getting started this weekend

1. Fetch the site's `/sitemap.xml` — that's your liberation manifest.
2. Have the assistant crawl five pages into Markdown with frontmatter.
3. `npx create-next-app@latest`, render one page, push to GitHub, open the Vercel preview next to the live Wix site.
4. Then: *"liberate the rest, starting with the top-20 traffic pages."*

The future CMS isn't a dashboard. It's a conversation — with git as the memory and the edge as the server.

---

*Fourth in the Web 4.0 Contrivance Engineering series. Built the way it describes: written with an AI assistant, versioned on GitHub, deployed on Vercel.*
