# Goodbye WordPress: Build and Run Websites on Next.js + GitHub + Vercel, with an AI Assistant as Your CMS

WordPress powers a huge share of the web, but for many sites it has become the heaviest possible way to do the simplest possible job: publish pages. PHP hosting, plugin updates, security patches, database backups, a wp-admin panel that loads slower every year — all to serve what is, in the end, mostly static content.

There is a simpler architecture now, and it is fully in reach of any working web developer:

- **Next.js** renders the site (static where possible, dynamic where needed)
- **GitHub** is the database — content lives in Markdown files, versioned like code
- **Vercel** hosts it, with a preview deployment for every change
- **An AI assistant** (like Muse) is the CMS — it writes, edits, organizes, and publishes content through plain conversation

No wp-admin. No plugin updates. No 3 a.m. "your site was defaced" emails. This guide shows you how to build a new site this way, how to migrate an existing WordPress site onto it, and — in Part 5 — how to pitch it to new clients choosing a stack, and to existing WordPress site owners.

---

## The frame: Web 4.0 Contrivance Engineering

This guide is a migration playbook, but the idea underneath it is bigger — a rethinking of what content management actually needs to be in the age of AI. Call it **Web 4.0 Contrivance Engineering**.

The web's eras, compressed: Web 1.0 gave us static pages. Web 2.0 gave us the dynamic, database-driven site — and WordPress rode that wave to power a huge share of the internet. Web 3.0 chased decentralization. Web 4.0 is the intelligent web: AI agents as the interface layer, with version control as the memory and the edge as the infrastructure.

**Contrivance engineering** is the discipline of replacing an entire software category with a smaller, cleverer mechanism — a contrivance. The CMS is the first casualty, because a CMS was only ever two jobs:

1. **Store content with structure.** Git does this better than MySQL ever did: every version, every author, every change, diffable forever.
2. **Give non-technical people a way to change it.** The dashboard was the answer in 2005. The better answer now is language itself — you describe the change, the assistant makes it, you review the result.

Content management was never really about the dashboard. It was always about the loop: *intent → change → review → publish.* WordPress built a cathedral around that loop — PHP, MySQL, plugin markets, security patches. The contrivance rebuilds it from four cheap parts: a chat, a branch, a preview URL, and a merge.

Everything that follows is that contrivance, engineered.

---

## Part 1 — The architecture

### How the pieces fit

```
You  →  (chat)  →  AI assistant  →  edits Markdown/MDX in GitHub repo
                                              ↓
                                    Vercel builds a preview URL
                                              ↓
                              You review → merge → production deploys
```

**Content** lives in the repo as Markdown (or MDX, Markdown with React components) files with frontmatter:

```markdown
---
title: "Why we moved off WordPress"
date: "2026-10-03"
author: "Andrew"
tags: ["nextjs", "wordpress", "cms"]
description: "How we replaced wp-admin with an AI assistant."
---

Your article body here. **Markdown** works exactly as you'd expect,
and MDX lets you drop in interactive components:

<Callout>Like this one.</Callout>
```

**GitHub** stores every version of every page. The full history of your site's content is a `git log` away — something WordPress revisions never did well.

**Vercel** watches the repo. Push to `main` → production deploys in under a minute. Open a pull request → you get a unique preview URL to review the change on a live site before it ships.

**The AI assistant** replaces wp-admin. Instead of logging into a dashboard, you say:

- *"Write a blog post about our new pricing, friendly tone, 600 words"*
- *"Update the homepage hero to mention the October sale"*
- *"Add these three testimonials to the reviews page"*
- *"The About page photo is outdated — swap in the new team shot and optimize it"*

The assistant edits the files, commits to a branch, and hands you a preview link. You approve, it merges, the site updates. That loop *is* content management now.

### Why this beats WordPress for content sites

| | WordPress | Next.js + GitHub + Vercel + AI |
|---|---|---|
| Hosting cost | $5–50/mo PHP hosting | $0 (Vercel hobby tier is generous) |
| Security surface | PHP + MySQL + 20 plugins, all patchable | Static files on a CDN; nothing to hack |
| Speed | Often 2–5s TTFB | <1s, global edge CDN |
| Content history | Fragile revisions table | Full git history, branches, diffs |
| Editing UX | wp-admin dashboard | A conversation |
| Staging | Plugin or manual copy | Automatic preview URL per change |

---

## Part 2 — Migrating an existing WordPress site

### Step 1: Audit what you actually have

Before touching code, inventory the site. Most WordPress sites are 90% pages and posts, plus a handful of dynamic bits.

1. **Content inventory** — list every post type: posts, pages, custom post types, media. Tools → Export in wp-admin gives you the shape of it.
2. **Plugin audit** — for each active plugin, write down what it *does*, not what it's called. "Contact Form 7" → "contact form". "Yoast" → "meta tags + sitemap". "WooCommerce" → stop here; a full store migration is a bigger project (see the tradeoffs section).
3. **URL inventory** — export your permalink structure. You'll need it for redirects in Step 5.
4. **Traffic inventory** — note your top 20 pages in analytics. Those get migrated first and tested hardest.

### A design decision first: convert the theme, or redesign?

Right after the audit you'll hit a fork: rebuild the current theme faithfully in Next.js, or use the migration as the moment to redesign. Both are valid — pick deliberately.

**Option A — faithful theme conversion.** Recreate the existing look with Next.js components and Tailwind. Fastest path, zero brand disruption, and stakeholders see "the same site, but instant." Map WordPress templates to routes as you go:

| WordPress template | Next.js equivalent |
|---|---|
| `header.php` / `footer.php` | `app/layout.tsx` |
| `front-page.php` | `app/page.tsx` |
| `single.php` | `app/blog/[slug]/page.tsx` |
| `archive.php` | `app/blog/page.tsx` |
| `page.php` | `app/[slug]/page.tsx` or per-page routes |

Hand the assistant the theme's CSS — or screenshots of each template — and have it generate the Tailwind equivalents. This is mechanical work it's fast at.

**Option B — redesign during the migration.** The content is already being restructured and redirects mapped, so this is the cheapest redesign moment you'll ever get. Go content-first: fix the information-architecture problems the audit found, keep URL structure stable where you can, and let every design iteration ride the preview-URL workflow from Step 7.

**The pragmatic rule:** if the site has real traffic, convert faithfully first — isolate the platform risk — then redesign on the new stack, where every change gets its own preview link. If traffic is small, combine them and skip the double work.

### Step 2: Export content and convert it to Markdown

The cleanest path is the WordPress REST API or WP-CLI:

```bash
# Export posts via WP-CLI (run on the server or a local copy)
wp export --dir=./wp-export --post_type=post,page
```

The export is XML (WXR format). Convert each item to a Markdown file with frontmatter. This is exactly the kind of tedious transformation an AI assistant does well — hand it the export and say *"convert every post to Markdown with title, date, slug, and categories in frontmatter, HTML converted to Markdown, images referenced from /images/"*. It can write and run the conversion script for you.

For smaller sites, even simpler: the assistant can crawl the live site's REST API (`/wp-json/wp/v2/posts?per_page=100`) and generate the Markdown files directly.

**Media:** download `/wp-content/uploads/` and move it into the new repo (or object storage for large libraries). Rename files to something sane along the way.

### Step 3: Scaffold the Next.js app and content layer

```bash
npx create-next-app@latest my-site --typescript --tailwind --app
cd my-site
npm install gray-matter remark remark-html
```

A minimal content layer — `lib/posts.ts`:

```ts
import fs from "fs";
import path from "path";
import matter from "gray-matter";
import { remark } from "remark";
import html from "remark-html";

const postsDir = path.join(process.cwd(), "content/posts");

export function getAllPosts() {
  return fs.readdirSync(postsDir)
    .filter(f => f.endsWith(".md") || f.endsWith(".mdx"))
    .map(file => {
      const { data } = matter(fs.readFileSync(path.join(postsDir, file), "utf8"));
      return { slug: file.replace(/\.mdx?$/, ""), ...data };
    })
    .sort((a: any, b: any) => +new Date(b.date) - +new Date(a.date));
}

export async function getPostHtml(slug: string) {
  const raw = fs.readFileSync(path.join(postsDir, `${slug}.md`), "utf8");
  const { content } = matter(raw);
  return (await remark().use(html).process(content)).toString();
}
```

Then a dynamic route at `app/blog/[slug]/page.tsx` using `generateStaticParams()` to pre-render every post at build time. (For MDX with components, swap in `next-mdx-remote` — same idea, more power.)

Your repo layout ends up looking like this:

```
my-site/
├── app/                  # routes: /, /blog, /blog/[slug], /about ...
├── content/
│   ├── posts/            # one Markdown file per post
│   └── pages/            # homepage, about, pricing ...
├── public/images/        # migrated media
└── lib/posts.ts          # content helpers
```

### Step 4: Rebuild the dynamic bits without plugins

Every WordPress plugin maps to something simpler:

- **Contact forms** → a React form + Next.js Server Action, sending via Resend or Formspree. No plugin, no spam-filter plugin, no database table.
- **SEO meta + sitemaps** → Next.js Metadata API plus an auto-generated `sitemap.ts` and `robots.ts`. The assistant writes these once; they stay correct forever.
- **Search** → build-time index with Pagefind — fast, offline-capable, zero backend.
- **Comments** → Giscus (GitHub Discussions as the backend — fitting, since your content already lives on GitHub) or Disqus if you prefer managed.
- **Newsletters** → Buttondown, ConvertKit, or Substack embed.
- **Analytics** → Vercel Analytics or Plausible. No cookie-banner plugin circus.

### Step 5: Redirects — don't torch your SEO

First principle: the best redirect is the one you never need. **Retain the original URLs wherever possible** — carry WordPress slugs over verbatim as your Next.js slugs. Every redirect leaks a little link equity and adds a failure point, so only redirect URLs you're deliberately changing (like collapsing date-based permalinks into clean slugs). Resist the urge to "improve" slugs during the migration; do that later, one URL at a time, once the new stack is proven.

For the URLs that do change, make the old ones keep working. WordPress permalinks like `/2024/05/my-post/` need to resolve. In `next.config.ts`:

```ts
const nextConfig = {
  async redirects() {
    return [
      // date-based permalinks → clean slugs
      { source: "/:year(\\d{4})/:month(\\d{2})/:slug", destination: "/blog/:slug", permanent: true },
      // renamed pages
      { source: "/about-us", destination: "/about", permanent: true },
    ];
  },
};
```

Generate the full redirect map from your URL inventory (Step 1) — another job to hand the assistant: *"here's the CSV of old URLs and new slugs, generate the redirects array."*

### Step 6: Deploy on Vercel

1. Push the repo to GitHub.
2. Import it in Vercel — it auto-detects Next.js. Zero config for most sites.
3. Point your domain's DNS at Vercel; it provisions HTTPS automatically.
4. Set a freeze on the WordPress site, do a final content export, merge, switch DNS.

Keep the old WordPress export archived in the repo (`/archive/wordpress-export.xml`) — it's your ultimate backup, and it costs nothing to keep.

### Step 7: Set up development, staging, and production branches

WordPress gave you one live site and crossed fingers. Git gives you three environments for free:

| Branch | Environment | Deploys to |
|---|---|---|
| `main` | **Production** | Your domain, via Vercel production deployment |
| `staging` | **Staging** | A staging URL (e.g. `staging.yoursite.com`) that mirrors production |
| Feature branches (`post/...`, `fix/...`) | **Development** | Automatic Vercel preview URLs — one per pull request |

**How it works on Vercel:** set `main` as the production branch. Every push to `staging` gets its own stable deployment — point a `staging` subdomain at it and password-protect it under Vercel's Deployment Protection settings. Every pull request, from any branch, gets a throwaway preview URL for review.

**The workflow, with the AI assistant in the loop:**

1. You ask for a change; the assistant works on a short-lived branch (`post/weekend-hours`).
2. It opens a pull request **against `staging`**, not `main`. Vercel builds a preview; you review the change on the staging site, in context with everything else in flight.
3. When staging looks right, merge `staging` → `main`. Vercel promotes it to production. Done.

**Protect the important branches** (GitHub → Settings → Branches → Add branch protection rule, for both `main` and `staging`):

- Require a pull request before merging — no direct pushes, not even by you
- Require at least one approval — yours is enough to keep the assistant honest
- Require status checks to pass — the Vercel build must be green before merge

This is the guardrail that makes "AI as CMS" safe: the assistant can draft anything, but only you move it up the ladder — development → staging → production.

### Don't skip: the pitfalls list

- **Shortcodes.** `[gallery]`, `[contact-form]`, and friends don't survive HTML→Markdown conversion as anything useful. Inventory them in Step 1 and rebuild each as an MDX component or a plain section.
- **Email — the big one.** When you point DNS at Vercel, copy *every* DNS record first, especially **MX records**. Teams have migrated a site in an afternoon and killed company email for a week. Export the zone file before you touch anything.
- **RSS.** If anyone subscribes at `/feed/`, keep it working — generate an RSS route in Next.js or redirect the old feed URL to it.
- **Drafts and scheduled posts.** Exports grab published content. Decide what happens to drafts and scheduled items *before* cutover, or they'll silently vanish.
- **OG images, Twitter cards, favicon.** Regenerate and test with a link-preview checker; they're the first thing people notice when they're wrong.
- **Test the forms.** Submit every form on staging and confirm the notification actually arrives. This is the #1 "we launched and nobody told us" failure.

### Step 8: Cutover day checklist

1. **Content freeze** on WordPress — no new posts or edits after an agreed time.
2. **Final export** → convert → merge to `staging`. Review *everything* on the staging URL, especially the top-20 traffic pages.
3. **Merge `staging` → `main`**, then switch DNS (Vercel's A/CNAME records — with MX/TXT records preserved per the pitfalls list).
4. **Search Console:** submit the new sitemap, request indexing on key pages.
5. **Watch 404s for two weeks** (Vercel logs or a tiny script) and add redirects for anything missed.
6. **Keep the old WordPress site read-only for 30 days** — a live reference and an instant rollback. Rollback is just DNS pointed back at the old host.

---

## Part 3 — The AI assistant as your CMS

This is the part that changes the job, not just the stack. Once the site is Git-backed, "content management" becomes a conversation. Here's what the daily workflow looks like:

### Publishing a post

> **You:** "Write a post announcing our new weekend hours — Saturdays 9 to 5 starting October 11. Friendly tone, mention the coffee's on us that day."
>
> **Assistant:** drafts `content/posts/weekend-hours.md` with frontmatter (title, date, description for SEO), commits to a branch, Vercel builds a preview.
>
> **You:** open the preview URL on your phone, reply "make the headline punchier."
>
> **Assistant:** revises, preview rebuilds in ~30 seconds.
>
> **You:** "Ship it." → merged → production.

No login, no editor quirks, no "where did the publish button go after the update."

### Things the assistant does that wp-admin never could

- **Bulk operations in one sentence:** *"Add a 'last updated' note to every post from 2023"* or *"retitle all posts to title case."*
- **SEO hygiene on autopilot:** *"Check every page for missing meta descriptions and draft them."*
- **Broken-link patrol:** *"Crawl the site and fix or flag every broken link."*
- **Image work:** resize, convert to WebP/AVIF, write real alt text, swap assets — *"optimize every image over 500KB."*
- **Redirects and sitemaps:** regenerated from the actual content, never stale.
- **Translations:** *"Translate the pricing page to French"* — a new locale route in minutes.
- **Institutional memory:** it reads the whole repo, so *"make the new landing page match the tone of our March campaign"* actually works.

### Guardrails (because "AI, just publish" needs brakes)

1. **Branch + preview for everything.** The assistant works on short-lived branches; nothing reaches `staging` without a preview URL you've seen, and nothing reaches `main` without passing through `staging` (see Step 7).
2. **Pull-request review.** Treat content like code: the PR diff *is* the editorial review. Branch protection rules can require your approval before merge.
3. **Keep humans on facts and voice.** The assistant drafts fast, but prices, dates, claims, and tone get your eyes before they go live.
4. **Everything is reversible.** A bad publish is `git revert` away — try doing that with a WordPress auto-update gone wrong.

### The AI CMS prompt library — steal these

The workflow only works if you know what to ask for. Copy-paste starters, grouped by job:

**Publishing**
- *"Draft an 800-word post on [topic] for [audience]. Frontmatter: title, today's date, a sub-160-character meta description, and three tags from our existing tag list. Open a PR against staging."*
- *"Turn these meeting notes into a changelog entry — keep it under 150 words: [paste]."*

**SEO & hygiene**
- *"Audit every file under /content for missing or over-160-character meta descriptions. Draft replacements in our voice and open one PR."*
- *"Find duplicate titles across posts and propose rewrites that differentiate them."*

**Maintenance**
- *"Find every image over 400KB, convert to WebP, update the references, and report total bytes saved."*
- *"Check all external links across the site. For each dead one, suggest a replacement or flag it for me."*
- *"Regenerate the sitemap from current content and confirm every URL returns 200 on the staging deployment."*

**Repurposing**
- *"Turn this post into a 5-part thread outline and a 100-word newsletter blurb: [slug]."*
- *"Summarize our three most-visited posts into a new 'Start here' page."*

---

## Part 4 — Honest tradeoffs: when WordPress still wins

This stack isn't for everyone. Be straight with clients (or yourself) about the limits:

- **Non-technical teams** who live in a visual page builder (Elementor, Divi) will miss drag-and-drop. The AI-chat workflow replaces the *blogging* use case well, but not the *design canvas* use case.
- **Complex membership / LMS / marketplace functionality** (MemberPress, LearnDash, Dokan) has no one-line Next.js equivalent. That's application development, not a migration.
- **WooCommerce stores** can move to Shopify/Stripe + Next.js storefronts, but it's a replatforming project with real cost — don't pitch it as a weekend migration.
- **Someone has to own the repo.** The AI assistant does the work, but a human still reviews PRs. If the client won't click "merge," nothing ships.

For brochure sites, blogs, portfolios, docs, and marketing sites — which is most of WordPress's footprint — the tradeoff is overwhelmingly in favor of the new stack.

---

## Part 5 — Pitching it: selling AI-as-CMS to WordPress clients

Parts 1–4 are the *how*. This is the *why* — written for the two WordPress conversations you'll actually have: a new client choosing a stack, and an existing WordPress owner deciding whether to migrate.

### New clients: WordPress quote vs. the contrivance

Don't pitch technology; pitch the absence of chores. They're comparing your quote against a WordPress build — hosting fees, maintenance retainer, plugin updates, and "your site got hacked" risk. Your pitch:

- **"You'll never log into a dashboard."** Content changes happen by messaging — you, or the AI assistant directly. They describe what they want, get a preview link, approve, it's live. No training session on wp-admin required.
- **No maintenance retainer for busywork.** There's no PHP to patch and no plugins to update. If you sell a care plan, it's for *content operations and improvements* — work they can see — not invisible janitorial work.
- **They own everything.** The site lives in *their* GitHub repo as plain Markdown and React. If you get hit by a bus, any developer on earth can take over — no tribal WordPress knowledge required.

Price the build normally; the differentiator is that the ongoing story is simpler and cheaper than the WordPress quote sitting next to yours.

### Existing WordPress owners: the migration pitch

These clients don't need the concept sold — they've lived the pain: update anxiety, security scares, 45-second wp-admin loads, the plugin renewal emails. Your pitch is relief with no sacrifice:

- **You keep everything that matters.** Every post, page, image, and URL — plus the design, rebuilt faithfully. Nothing your visitors love changes.
- **You lose everything that hurts.** No more core and plugin updates, no security patching, no backup plugins, no renewal stack. The attack surface goes from "all of PHP" to "static files."
- **The migration itself is low-risk.** They review the entire new site on a staging URL before anything goes live, DNS rollback takes minutes, and the old site stays read-only for 30 days as a safety net.

Point them at Parts 1–4 of this article. Developers respect a migration plan they can read.

### Handling the two objections you'll hear

**"What if the AI gets it wrong?"** Nothing publishes without approval. The assistant drafts on branches, the client reviews a preview link, merges go through staging. It's *more* controlled than the WordPress workflow, where anyone with an editor login can hit Publish.

**"What if you disappear?"** Everything is in their GitHub account — code, content, full history. The content is Markdown, the hosting is Vercel, the "CMS" is a documented conversation workflow. No license keys, no proprietary lock-in, no single point of failure with your name on it. That's a trust-builder most freelancers can't offer.

### The cost story, in one table

| | Typical WordPress, per year | The contrivance |
|---|---|---|
| Hosting | $60–600 | $0 (Vercel hobby; ~$240/yr Pro when you outgrow it) |
| Premium plugins/themes | $100–400 in renewals | $0 |
| Maintenance retainer | $1,200–6,000 | $0 of busywork |
| Security incidents | Unbounded | Near-zero attack surface |

The build costs what a build costs. It's the *second* year that sells it.

---

## Getting started this weekend

1. `npx create-next-app@latest` — scaffold it.
2. Export one WordPress post, convert it to Markdown, render it at `/blog/[slug]`.
3. Push to GitHub, import into Vercel, open the live URL.
4. Then hand your AI assistant the WordPress export and say: *"migrate the rest."*

The future CMS isn't a dashboard. It's a conversation — with git as the memory and the edge as the server.

---

*Built the way it describes: written with an AI assistant, versioned on GitHub, deployed on Vercel.*
