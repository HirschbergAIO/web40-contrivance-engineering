# Web 4.0 Contrivance Engineering

An article series for web developers on replacing the CMS with a smaller, cleverer mechanism: **Next.js + GitHub + Vercel, with an AI assistant as the content manager.**

The thesis: a CMS was only ever two jobs — *store content with structure* (git does it better than any database) and *give non-technical people a way to change it* (language beats dashboards). Content management was never about the dashboard; it was always the loop — *intent → change → review → publish* — rebuilt here from four cheap parts: a chat, a branch, a preview URL, and a merge.

## The articles

Each article is a self-contained page (open the folder — the `index.html` renders the article; the `README.md` holds the same text in markdown).

| # | Article | Migration story |
|---|---------|-----------------|
| 1 | [Goodbye WordPress](wordpress-to-nextjs-ai-cms/) | Replace wp-admin with an AI assistant; the flagship migration playbook |
| 2 | [Goodbye FTP](static-html-to-nextjs-ai-cms/) | Unfreeze static HTML sites; install a CMS where none existed |
| 3 | [Goodbye App Stack](shopify-to-nextjs-ai-cms/) | Unbundle Shopify: headless Next.js storefront, AI merchandising |
| 4 | [Goodbye Lock-in](wix-to-nextjs-ai-cms/) | Escape Wix: the AI jailbreaks content with no export to work from |
| 5 | [Goodbye Template Cage](squarespace-to-nextjs-ai-cms/) | Keep the beauty, lose the cage |
| 6 | [Goodbye Webflow](webflow-to-nextjs-ai-cms/) | Graduate to code you own |

Every article follows the same skeleton: the frame, the architecture, an 8-step migration playbook, the AI-as-CMS workflow with a steal-these prompt library, honest tradeoffs, and a pitching guide for clients.

## Publishing

This site is published with GitHub Pages — fittingly, each article page is a single self-contained HTML file, no build step.

*Written with an AI assistant. The series practices what it preaches.*
