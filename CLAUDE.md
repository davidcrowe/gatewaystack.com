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
