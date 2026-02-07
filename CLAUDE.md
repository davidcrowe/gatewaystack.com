# gatewaystack.com

## What This Is

Marketing site for GatewayStack — secure user-scoped trust and governance for AI systems using OpenAI Apps SDK and MCP.

**Positioning:** B2B infrastructure layer for enterprise AI deployments. The primitives every agent ecosystem will eventually rely on.

## Architecture

GatewayStack has 6 composable modules:

| Module | Domain | Purpose |
|--------|--------|---------|
| **identifiabl** | identifiabl.com | Bind every request to a real user and context |
| **transformabl** | transformabl.com | Safety, normalization, classification, preprocessing |
| **validatabl** | validatabl.com | Permissions, policies, roles, organizational rules |
| **limitabl** | limitabl.com | Rate limits, quotas, budgets, spend controls |
| **proxyabl** | proxyabl.com | Identity-aware routing across models and providers |
| **explicabl** | explicabl.com | Full audit trail, traces, observability |

Individually useful but designed to interlock.

---

## Stack

- Jekyll (static site generator)
- Markdown content
- Deployed to GitHub Pages

## Key Files

- `_layouts/default.html` — Main layout with Plausible analytics, styles
- `index.md` — Homepage with product pitch and architecture overview
- `writing.md` — Blog index
- `_posts/*.md` — Blog posts
- `_config.yml` — Jekyll configuration

## Commands

```bash
bundle install           # install dependencies
bundle exec jekyll serve # local dev server (localhost:4000)
bundle exec jekyll build # production build to _site/
```

## Analytics

- Plausible (self-hosted at analytics.reducibl.com)
- Domain: gatewaystack.com
- Events: Outbound Link: Click (GitHub, LinkedIn, module sites)

## Key CTAs

- **GitHub:** `https://github.com/davidcrowe/gatewaystack` — "inspect the code"
- **Enterprise contact:** Links to reducibl for enterprise deployments
- **LinkedIn:** Follow for updates
- **Module sites:** Links to each *-abl module

## Notes

- never add co-authored-by to commits
- B2B positioning — enterprise focus, technical audience
- periwinkle (#E1E6F8) background matches reducibl brand
- no waitlist — direct to GitHub or enterprise contact

## Build Validation

Always verify after making changes:
1. `bundle exec jekyll build` — must succeed with zero errors
2. Check `_site/` output for expected pages
3. Validate HTML structure renders correctly

## Deployment Pipeline

Push to main → GitHub Pages auto-builds Jekyll → site live at gatewaystack.com.
- CNAME file maps to gatewaystack.com
- No custom CI/CD — relies on GitHub Pages built-in Jekyll support
- `_site/` is generated output, never committed

## Language & Content Conventions

- This is a pure Jekyll/Markdown site — no TypeScript, no npm, no build tooling beyond Ruby/Bundler
- Kramdown markdown processor
- Minima theme (customized in _layouts/default.html)
- Periwinkle (#E1E6F8) background matches reducibl brand identity
- B2B enterprise positioning — technical audience, no consumer language
- All headings and titles use lowercase aesthetic

---

## git workflow

- `main` — auto-deploys to GitHub Pages. Only push verified content.
- `dev/<feature>` — for multi-step content changes
- **Commit early and often** — after each meaningful edit
- **No Co-Authored-By: Claude** — never include Claude attribution
- Always run `bundle exec jekyll build` before pushing to main
