# Goodbye Webflow: Graduate to Code You Own on Next.js + Vercel, with an AI Assistant as Your CMS

Let's be clear up front: Webflow is the best of the website builders, and it isn't close. The Designer is a genuinely great tool, the code export is the cleanest in the industry, and CMS Collections taught a generation of marketers to think in structured content. This article isn't an escape plan. It's a graduation plan.

If you build in Webflow, you already think in components, classes, and collections. Next.js is the same mental model with no ceiling: own the code, drop the per-seat taxes, and replace the Editor with an AI assistant that does content operations in conversation. This is the sixth article in the Web 4.0 Contrivance Engineering series — same destination as the others (Next.js on GitHub + Vercel, AI assistant as CMS), tuned for the starting point with the smoothest on-ramp in the series.

---

## The frame: graduation, not escape

The series thesis, compressed: **contrivance engineering** is replacing an entire software category with a smaller, cleverer mechanism. A CMS was only ever two jobs — **store content with structure** (git does it better than anything) and **give non-technical people a way to change it** (language beats dashboards). The loop was always *intent → change → review → publish*; the contrivance rebuilds it from four cheap parts: a chat, a branch, a preview URL, and a merge.

Here's the thing Webflow people grasp faster than anyone: you already live next door to the contrivance. Symbols are components. CMS Collections are structured content with a schema. Publishing is a deploy. Graduating just means owning the mechanism instead of renting it — the collections become Markdown files, the Designer becomes code, and the Editor becomes a conversation. Nothing about how you *think* changes. Only who holds the keys.

---

## Part 1 — The architecture

Same four pieces as every article in the series:

- **Next.js** renders the site (static where possible, dynamic where needed)
- **GitHub** is the database — content lives in Markdown/MDX files, versioned like code
- **Vercel** hosts it, with a preview deployment for every change
- **An AI assistant** is the CMS — content operations happen in plain conversation

What's different for Webflow: **the migration starts from Webflow's code export, so this is the most mechanical migration in the series** — closest in spirit to the static-HTML article, but with a head start. Your symbols map 1:1 to components. Your CMS Collections map to Markdown files with a frontmatter schema. Your classes come along in the exported CSS. The design doesn't get "rebuilt" — it gets *carried over*, pixel-faithful, on day one.

```
Webflow Designer  →  code export  →  Next.js components + Markdown
CMS Collections   →  CSV / API   →  content/ with frontmatter
Editor            →  (retired)   →  AI assistant + preview URLs
```

---

## Part 2 — The migration playbook

### Step 1: Audit what you have

Webflow sites are well-organized by construction, so this goes fast:

1. **Pages** — list them; note the CMS Collection Pages (blog posts, case studies) separately from static pages.
2. **CMS Collections** — for each collection (Blog, Work, Team…), list its fields. These become your frontmatter schema in Step 4. Note reference and multi-reference fields — they become frontmatter arrays or relational lookups.
3. **Symbols / components** — your nav, footer, CTAs. These map 1:1 to Next.js components.
4. **Interactions inventory** — list every IX2 interaction and animation. Simple scroll/fade effects are trivial to carry over; complex multi-step interactions are the fiddliest part of this migration (Step 5).
5. **Forms, Logic, Memberships** — any Logic flows, form notification endpoints, or gated member content need a rebuild decision (Step 6).

### Step 2: Export — code and content

Webflow's export story is the best in the builder world, and you get two exports:

- **Code export** (HTML/CSS/JS, from the Designer) — your pages, your classes, your interactions runtime. This is the raw material for Step 3.
- **CMS content** — per-collection CSV export from the CMS panel, or the CMS API for larger or frequently-updated collections. Rich Text fields come along as HTML, which converts cleanly to Markdown.

Hand both to the AI assistant. This is mechanical transformation work — exactly what it's fast at — and it can write the conversion scripts for collections with hundreds of items.

### Step 3: Componentize the export

Same technique as the static-HTML article's Step 2, but easier: Webflow already did the component thinking for you.

1. Scaffold the Next.js app (`npx create-next-app@latest`).
2. Give the assistant the exported HTML: *"Convert each Symbol to a Next.js component (Nav, Footer, CTA…), pages to routes, and keep the exported CSS and class names intact."*
3. Drop the exported stylesheet in as-is on day one — it works, and the design arrives pixel-faithful. Migrate to Tailwind later, component by component, only if you want to.

Because the export is machine-generated, there's almost no drift to reconcile — the "flag the differences" step that mattered for hand-coded sites barely applies here.

### Step 4: Collections become Markdown

This is the satisfying part. A Webflow CMS Collection already *is* a content model; you're just changing the storage format. Collection fields become frontmatter:

```markdown
---
title: "Q3 Brand Refresh"
slug: "q3-brand-refresh"
date: "2026-07-14"
client: "Acme Co"              # was a Reference field
services: ["Identity", "Web"]  # was a multi-reference field
cover: "/images/work/q3-refresh.jpg"
summary: "Full rebrand and marketing site."
---

Rich text body, converted from the collection's Rich Text field…
```

One Markdown file per collection item, in `content/work/`, `content/blog/`, etc. Collection Pages become dynamic routes (`app/work/[slug]/page.tsx`) with `generateStaticParams()`. Reference fields resolve at build time — the assistant generates the lookup code from your field list.

### Step 5: Interactions — keep, then rebuild

Be honest with yourself here: this is the fiddliest step.

- **Keep what works:** the exported `webflow.js` powers IX2 interactions. For simple scroll, fade, and hover effects, ship it as-is on day one. It works.
- **Rebuild what matters:** signature animations worth keeping long-term get rebuilt with framer-motion (or CSS) as proper React — cleaner, lighter, and yours.
- **Triage the rest:** complex multi-step IX2 timelines are where migrations stall. Decide per interaction: keep via webflow.js, rebuild, or cut. Most sites have three interactions that matter and thirty nobody will miss.

### Step 6: Forms, Logic, and Memberships

- **Forms** → a React form + Next.js Server Action via Resend or Formspree. Rebuild the notification routing (who gets emailed) that Webflow handled.
- **Logic flows** → evaluate each one. Simple automations (form → email → CRM) become Server Actions or a Make/Zapier step. Complex multi-step Logic is application work — scope it honestly.
- **Memberships / gated content** → this is the one place to slow down. Clerk or Auth.js handles it well in Next.js, but if Memberships is load-bearing, consider keeping Webflow serving the gated section during a transition period. Don't migrate the paywall on a deadline.

### Step 7: Deploy — GitHub + Vercel, branches included

Push to GitHub, import into Vercel, point DNS. Set up the series-standard development → staging → production branch ladder: feature branches get preview URLs, `staging` is the review environment, `main` is production, branch protection rules on both. The assistant works on branches; nothing ships without a preview you've seen.

### Step 8: Cutover + the pitfalls list

1. Content freeze in Webflow — no Editor changes after an agreed time.
2. Final CMS export → convert → merge to `staging`. Review everything on the staging URL.
3. Merge `staging` → `main`, switch DNS.
4. Search Console: submit the sitemap, request indexing on key pages.
5. Keep the Webflow site (downgraded to a cheaper plan or paused) for 30 days as instant rollback.

**Pitfalls:**

- **Class-name collisions.** Webflow's exported CSS uses generated class names that can collide with your own or with Tailwind utilities. Namespace or scope the export stylesheet on day one.
- **Responsive images.** Webflow auto-generates `srcset`s — make sure the exported image variants actually ship with the export, and consider moving to `next/image` for the important ones.
- **Form notification endpoints.** Rebuild *and test* who gets notified — the #1 "we launched and nobody told us" failure, same as every article in this series.
- **CMS API rate limits.** Large collections pulled via API need paged, throttled scripts — the assistant writes these, but verify the full item count matches.
- **Email, as always.** Copy every DNS record before switching — MX records especially.

---

## Part 3 — The AI assistant as your CMS

This audience will love the prompt library — you already do content operations; now they're conversational. The daily loop: chat → branch → preview URL → approve → merge.

> **You:** "Add the new case study to the Work collection using the Q3 template — client is Northwind, services are Web and Motion. Draft the summary from these notes: [paste]."
>
> **Assistant:** creates `content/work/northwind.md` with the right frontmatter, commits to a branch, Vercel builds a preview.
>
> **You:** check the preview, "tighten the headline."
>
> **Assistant:** revises. **You:** "Ship it." → staging → production.

### The Webflow-graduate prompt library — steal these

- *"Add the new case study to the Work collection with the Q3 template: client [X], services [Y]. Draft the summary from these notes: [paste]."*
- *"Regenerate OG images for every blog post missing one, using the post title and our brand template."*
- *"Find every collection item with an empty summary field and draft one in our voice."*
- *"Update the pricing on the Services page and every case study that mentions the old rate."*
- *"Audit all external links across collections; fix or flag the dead ones."*
- *"Convert the three most-visited blog posts' inline images to next/image with real alt text."*
- *"Draft next month's changelog from these merged PR titles: [paste]."*
- *"Check that every Work item has a cover image under 400KB; compress the offenders to WebP."*

---

## Part 4 — Honest tradeoffs: when Webflow still wins

Respect where it's due — Webflow's visual builder is genuinely excellent, and this section is shorter than the WordPress equivalent for a reason:

- **If your team lives in the Designer daily** — designers shipping layout changes without developers, hitting no limits — don't fix what isn't broken. The contrivance serves teams that feel the ceiling, not teams thriving under it.
- **Per-seat pricing is the real enemy, not the tool.** Do the math on Editor seats vs. a migration before deciding; sometimes the honest answer is "stay one more year."
- **Don't migrate mid-campaign.** If a launch, rebrand, or funding announcement is within 6 weeks, freeze the migration talk until after.
- **Logic/Memberships parity costs.** If those are load-bearing, the migration includes real application work — scope and price it as such, not as a weekend project.

---

## Part 5 — Pitching it: you've already outgrown it

Webflow clients don't need "website" sold to them — they need *graduation* sold to them. They already think in components and collections, so speak their language:

- **"Own the code."** No more exporting around the platform. The design system lives in your repo, versioned, diffable, portable.
- **"No seat taxes."** Content changes stop costing per-editor-seat. Anyone you trust can message the assistant; approval stays with you.
- **"No CMS item limits."** Collections grow to whatever git can hold — which is effectively unbounded.
- **"AI content ops."** The prompt library in Part 3 does in sentences what used to take an Editor session.

Name the pains they actually feel: the CMS item cap they're approaching, the seat invoice that grows with the team, hosting costs scaling with traffic. Then the objections:

**"What if the AI gets it wrong?"** Nothing publishes without approval — branch, preview, staging, merge. More controlled than the Editor, where a published change is live instantly with no review step.

**"What if you disappear?"** Everything is in their GitHub account — code, content, full history. No proprietary lock-in; any Next.js developer on earth can take over.

---

## Getting started this weekend

1. Export your Webflow site's code and one CMS collection as CSV.
2. `npx create-next-app@latest` — scaffold it, drop in the exported CSS.
3. Hand the assistant the export: *"convert the homepage and nav symbol to components."*
4. Push to GitHub, import into Vercel, open the preview URL next to the live site — then *"migrate the rest."*

The future CMS isn't a dashboard. It's a conversation — with git as the memory and the edge as the server.

---

*Sixth in the Web 4.0 Contrivance Engineering series. Built the way it describes: written with an AI assistant, versioned on GitHub, deployed on Vercel.*
