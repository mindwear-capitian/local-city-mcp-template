# local-city-mcp-template

This is a **template repo**, not a running server. It's the starting point
for building `local-{city}-mcp` — an MCP server that gives an AI assistant
plain-English access to official, public, no-key data about one metro area.

**Before you touch any code, read [STANDARD.md](STANDARD.md).** It's the
spec this template implements: the hard rules, the tool contract, the
testing bar, and the licensing pattern. This README just tells you how to
use the template mechanically; STANDARD.md tells you why it's built this
way.

Reference implementation this was extracted from:
[local-austin-mcp](https://github.com/mindwear-capitian/local-austin-mcp)
(41 tools, live in production).

## Quick start

1. **Use this template** (GitHub's "Use this template" button, or `gh repo
   create local-{city}-mcp --template mindwear-capitian/local-city-mcp-template`).
2. **Replace every placeholder.** Every file that needs per-city values uses
   bare `{{TOKEN}}` markers (no leading `$` — that's deliberate, so they
   never collide with `${{ }}` GitHub Actions expression syntax in
   `.github/workflows/ci.yml`, which you should leave untouched). Find them
   all with:

   ```bash
   grep -rl '{{[A-Z_]*}}' --include='*.js' --include='*.json' --include='*.md' .
   ```

   | Token | Example | Where it's used |
   |---|---|---|
   | `{{CITY}}` | `Denver` | Display name, in prose |
   | `{{CITY_SLUG}}` | `denver` | Tool prefix, package name, repo name |
   | `{{CITY_SLUG_UPPER}}` | `DENVER` | Env var name (`LOCAL_DENVER_MCP_TIER`) |
   | `{{MAINTAINER_NAME}}` | `Jane Doe` | Attribution, package.json author |
   | `{{MAINTAINER_URL}}` | `https://janedoe.com` | Attribution, homepage |
   | `{{GITHUB_USER}}` | `janedoe` | Repo URLs |
   | `{{DEFAULT_LAT}}` / `{{DEFAULT_LNG}}` | `39.74`, `-104.99` | Default map center for the example NWS tool |

   A one-shot sed pass works fine for a first pass — just review the diff
   before committing, since a couple of tokens (like `{{CITY}}` inside
   prose sentences) are case-sensitive on purpose.

3. **Run it:**

   ```bash
   npm install
   npm start          # runs the MCP over stdio
   npm test           # unit tests
   npm run test:contract  # boots the server, calls every tool via the real MCP layer
   ```

4. **Build your first real tool.** See CONTRIBUTING.md — copy
   `tools/environment/nws-alerts.js` (works for any US city untouched once
   you fill in the placeholders) as your pattern reference, or
   `tools/meta/about.js` for the minimal shape.

5. **When it's real:** update the README's tool table, `Sources of Truth`
   table, and Architecture section (this template's README is a skeleton —
   flesh it out the way
   [local-austin-mcp's README](https://github.com/mindwear-capitian/local-austin-mcp#readme)
   does once you have more than a couple of tools).

6. **Get listed:** once `test:contract` is green in CI and you meet
   STANDARD.md §2, open a PR against
   [awesome-local-mcp](https://github.com/mindwear-capitian/awesome-local-mcp).

## What's already built for you

| File | What it gives every tool for free |
|---|---|
| `lib/register.js` | Central registration: default MCP annotations, error→friendly-frame wrapping, `structuredContent` auto-promotion, tier gating |
| `lib/retry.js` | `retryFetch()` with jittered backoff + `UpstreamError` + LLM-friendly error text ("the MCP is fine, the upstream is having a problem") |
| `lib/request-context.js` | Propagates the MCP request's `AbortSignal` into every downstream fetch via `AsyncLocalStorage`, so client-side cancellation actually cancels in-flight calls |
| `lib/output-schemas.js` | Shared Zod shapes (`searchShape`, `openObjectShape`, etc.) for `outputSchema` |
| `lib/geocode.js` | U.S. Census geocoder — free, no key, works nationwide |
| `lib/logger.js` | stderr + MCP logging-notification logger |
| `lib/tiers.js` | Optional `core`/`all` tool-tier gating for once you have 20+ tools |
| `tools/meta/about.js` | Minimal tool example + the required `about` capability tool |
| `tools/environment/nws-alerts.js` | A **real, working example tool** (National Weather Service alerts) — copy its shape |
| `test/mcp-all-tools.js` | The required contract test (STANDARD.md §6) |
| `.github/workflows/ci.yml` | Unit + contract test on Node 20 + 22 |

None of this is speculative — it's copied from a production server that
shipped 41 tools. Read `lib/register.js` and `lib/retry.js` top-of-file
comments for the reasoning if you're wondering why a piece exists.

## License

This template is Apache License 2.0 (see [LICENSE](LICENSE)) — use it, fork
it, build a commercial product on it. Fill in `NOTICE` and `TRADEMARK.md`
with your own name/marks before you ship; the placeholders in those files
are examples, not defaults you inherit.
