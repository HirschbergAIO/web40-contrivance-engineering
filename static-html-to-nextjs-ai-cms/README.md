# Goodbye FTP: Move Static HTML Sites to Next.js + GitHub + Vercel, with an AI Assistant as Your CMS

The static HTML site is the web's cockroach: small-business sites built in 2014, hand-rolled portfolios, "my nephew made it" sites. They still work. They're fast. They cost almost nothing to host. And they are completely, utterly frozen — because every text change means finding the developer, waiting a week, and getting a $75 invoice for swapping a phone number.

This is the second article in the Web 4.0 Contrivance Engineering series (the first covered WordPress). Same destination — Next.js on GitHub + Vercel, with an AI assistant as the CMS — but a different starting point and, in one important way, an easier migration. There is no database to escape. The content is already files. You can move one page at a time.

---

## The frame: installing a CMS where none existed

The series thesis, compressed: a CMS was only ever two jobs — **store content with structure** (git does it better than anything) and **give non-technical people a way to change it** (language beats dashboards). The contrivance rebuilds the loop — *intent → change → review → publish* — from four cheap parts: a chat, a branch, a preview URL, and a merge.

WordPress migrations *replace* a CMS. Static-site migrations *install* one where none existed. The pitch isn't "lose the dashboard" — it's "stop emailing a developer to change a sentence." Everything that follows is tuned for that starting point.

---

## Part 1 — The architecture

Same four pieces as the flagship article:

- **Next.js** renders the site
- **GitHub** is the database — pages and content as versioned files
- **Vercel** hosts it, with a preview deployment for every change
- **An AI assistant** is the CMS — content changes happen in conversation

Two things are different for static sites:

1. **The migration can be incremental.** No database means no big-bang cutover. Move one page, or one section, at a time — the strangler-fig pattern. The old site keeps serving everything you haven't moved yet.
2. **FTP dies on day one.** Even before a single page migrates, getting the site into git and deploying from Vercel is a pure upgrade: full history, preview URLs, one-click rollback. Most static sites are still deployed by dragging files into FileZilla in 2026. That ends now.

```
Old world:  edit file → FTP upload → pray → (no history, no preview, no undo)
New world:  chat → branch → preview URL → approve → merge → deployed
```

---

## Part 2 — The migration playbook

### Step 1: Audit the fossil

Static sites accumulate quirks. Inventory them before touching anything:

1. **Page inventory** — list every `.html` file. Note which ones are actually visited (analytics, if it exists) and which are dead weight from 2016.
2. **Find the copy-paste.** Open three pages and diff the header, nav, and footer by eye. On hand-built sites they're usually duplicated verbatim across every page — sometimes with drift (the phone number updated on 8 of 11 pages). Count the duplications; that's your componentization hit list.
3. **Asset inventory** — CSS files (one? twelve?), JS (jQuery version? plugins for sliders, lightboxes, carousels?), images (anyone still shipping 4MB PNGs?), fonts.
4. **Hidden backends.** The site may be "static" with a PHP contact form (`mail.php`) hiding next to the HTML, or server-side includes. Find them now.
5. **FTP/hosting reality.** Who hosts it? Do you have the FTP credentials? Is there even a backup?

### Step 2: Componentize — the big mechanical win

This is where the AI assistant earns its keep immediately. The duplicated header/nav/footer across N pages becomes one layout component. The workflow:

1. Scaffold the Next.js app (`npx create-next-app@latest`).
2. Hand the assistant three representative pages and say: *"Extract the shared header, navigation, and footer into a single layout component. Flag any drift between pages — content that differs and needs a human decision."*
3. That drift-flagging is the valuable part. The assistant finds the 8-of-11 phone numbers and asks which is correct — work that takes a human an afternoon of squinting.

Before → after, conceptually:

```
before:  index.html, about.html, contact.html ... (header/nav/footer pasted into each)
after:   app/layout.tsx  ← header/nav/footer live here, once
         app/page.tsx, app/about/page.tsx, app/contact/page.tsx  ← only unique content
```

One edit now propagates everywhere. That alone unfreezes half the site's maintenance burden.

### Step 3: Extract content into Markdown

With the layout componentized, pull the page copy out of the HTML into Markdown files with frontmatter — same content layer as the WordPress article (`content/pages/`, gray-matter, `generateStaticParams`). The assistant does the extraction mechanically: *"Convert about.html's main content to Markdown with title and description frontmatter."*

**The incremental option:** you don't have to do every page. Leave rarely-changed pages as-is (even as raw HTML routes) and convert only the pages that change often — pricing, hours, news, team. Migrate the pain first.

### Step 4: Modernize the asset pipeline — gently

- **CSS:** don't rewrite it on day one. Drop the existing stylesheet in as-is; it works. Migrate to Tailwind incrementally, component by component, when you're touching things anyway.
- **Images:** run them through `next/image` and convert the worst offenders to WebP/AVIF — *"find every image over 500KB"* is a one-sentence job for the assistant now.
- **JavaScript:** audit, don't reflexively rewrite. A working jQuery slider can stay a working jQuery slider. Replace things when they break or when you need the functionality to change — not for fashion.
- **Fonts:** self-host or keep the existing links; just make sure they're not blocking render.

### Step 5: Replace the hidden backends

That PHP mail script becomes a Next.js Server Action with Resend or Formspree — same as the WordPress playbook. Server-side includes become components (already done in Step 2). Anything else server-side gets a decision: rebuild, replace with a service, or drop.

### Step 6: URLs — the easiest redirect story in the series

Static sites usually have clean 1:1 URLs already (`/about.html` → `/about`, or keep the `.html`). Retain them verbatim — the series rule stands: the best redirect is the one you never need. If you're dropping `.html` extensions, add the redirect map; it's usually a dozen lines.

### Step 7: Deploy — GitHub + Vercel, branches included

Push to GitHub, import into Vercel, point DNS. Set up the same development → staging → production branch ladder as the WordPress article (feature branches get preview URLs, `staging` is the review environment, `main` is production, branch protection on both). The assistant works on branches; nothing ships without a preview you've seen.

### Step 8: Cutover — lower risk than WordPress

There's no database to freeze and no plugin ecosystem to replicate, so cutover is anticlimactic — which is the point:

1. Move the pages that matter, verify on staging, merge to `main`.
2. Switch DNS to Vercel (and yes — copy the MX records first, same pitfall as ever).
3. Keep the old hosting account for 30 days as instant rollback.
4. Cancel the old hosting once the dust settles. Pocket the savings.

### Don't skip: the pitfalls list

- **Character encoding.** Old sites in Latin-1/Windows-1252 with curly quotes will mojibake when moved to UTF-8. Convert and check.
- **The PHP form nobody mentioned.** There's always one. Test every form on staging.
- **Hardcoded absolute URLs** (`http://www.example.com/images/...`) scattered through the HTML — normalize to relative paths or the new domain.
- **The `.htaccess` rules.** Redirects, password-protected directories, custom error pages — read it before you abandon Apache.
- **Email, again.** MX records. Every time.

---

## Part 3 — The AI assistant as your CMS

For WordPress migrants the assistant *replaces* wp-admin. Here it *becomes* the CMS the site never had. The owner's new workflow for a change that used to mean "email the developer":

> **Owner:** "Our holiday hours are Dec 24–26, closed. Update everywhere hours appear."
>
> **Assistant:** finds every page mentioning hours (now that content is Markdown, this is trivial), updates them on a branch, opens a preview.
>
> **Owner:** checks the preview on their phone. "Looks right."
>
> **You (or the assistant):** merge → staging → production.

No FTP. No invoice for a phone number. No week of waiting.

### The static-site prompt library — steal these

- *"Find every page that mentions our old address and update it to [new address]. Open a PR against staging."*
- *"Add this announcement banner to the top of every page until [date]: [text]."*
- *"Our menu PDF changed — replace it everywhere it's linked and confirm no page 404s."*
- *"Audit the site for pages that haven't been updated in 2+ years and list them with their last-modified dates from git."*
- *"Convert the gallery page's inline images to next/image with proper alt text."*

---

## Part 4 — Honest tradeoffs: when to leave the fossil alone

- **A 3-page site that changes once a year** and the owner is happy emailing you? The migration still pays off eventually, but be honest that the ROI clock runs on change frequency. Quote accordingly.
- **Don't rewrite working JavaScript for fashion.** jQuery isn't a moral failing. Replace it when it blocks something, not because it's old.
- **Someone still has to approve merges.** If the owner won't click "approve" on a preview, the AI CMS stalls the same way any CMS stalls without an editor. Set that expectation up front.

---

## Part 5 — Pitching it: selling the unfreezing

Static-site owners don't complain about dashboards — they complain about *friction*. Every change is an email, a wait, and an invoice. Your pitch:

- **"Your website is frozen. I'll unfreeze it."** After the migration, text changes are a message, not a work order. They describe it, see a preview, approve — live in minutes.
- **"You'll stop paying developer rates for sentences."** The $75 phone-number swap goes away forever.
- **"Nothing your customers see changes unless you want it to."** Same design, same URLs, same Google rankings — the migration is invisible to visitors. All the change is behind the curtain: git history, preview links, instant rollback.
- **The hosting bill probably drops too.** That $15/mo cPanel account becomes Vercel's free tier.

Objection handling is the same as the WordPress article: nothing publishes without approval (safer than FTP, where anyone with credentials overwrites live files), and everything lives in *their* GitHub repo — no lock-in, any developer can take over.

---

## Getting started this weekend

1. `npx create-next-app@latest` — scaffold it.
2. Feed the assistant your homepage + two inner pages: *"extract the shared layout."*
3. Push to GitHub, import into Vercel, open the preview URL next to the live site.
4. Then: *"migrate the rest, starting with the pages that change most."*

The future CMS isn't a dashboard. It's a conversation — with git as the memory and the edge as the server.

---

*Second in the Web 4.0 Contrivance Engineering series. Built the way it describes: written with an AI assistant, versioned on GitHub, deployed on Vercel.*
