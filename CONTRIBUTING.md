# Contributing

This file covers two different audiences — pick the section that applies.

## A) You're building a NEW local-{city}-mcp from this template

You're not really "contributing to" this repo — you're using it as a
starting point. See the README's Quick Start for the placeholder-replacement
mechanics. The rules below still apply to what you build; they're the same
rules the reference implementation (`local-austin-mcp`) follows, extracted
into [STANDARD.md](STANDARD.md).

## B) You're contributing back to THIS template repo

Improvements to the shared `lib/` infrastructure, the example tools, or the
spec itself are welcome — e.g. a bug in `retryFetch`'s backoff, a clearer
`STANDARD.md` section, a better worked-example tool. Anything genuinely
city-specific belongs in a city's own repo, not here.

## Ground rules (apply to every local-{city}-mcp, hard constraints)

Full detail in [STANDARD.md](STANDARD.md) §2. Summary:

1. **No credentials, ever.** These packages are self-hosted by strangers via
   `npx github:`. Zero required API keys or logins.
2. **Only official / public sources.** No third-party aggregators, no
   AI-generated content presented as fact.
3. **Every response carries a `source_url`.**
4. **Fail soft** in composed/multi-section tools.
5. **No PII** beyond what the source itself already publishes.

## Setup

```bash
git clone https://github.com/{{GITHUB_USER}}/local-{{CITY_SLUG}}-mcp
cd local-{{CITY_SLUG}}-mcp
npm install
npm start          # runs the MCP over stdio
```

Node 20+.

## Adding a tool

A tool is a small module that exports an object with `name`, `description`,
`inputSchema` (zod), and an async `handler`. Look at `tools/meta/about.js`
for the minimal shape, and `tools/environment/nws-alerts.js` for a full
real-data-fetching example (geocoding, `retryFetch`, structured output,
graceful error handling).

1. Create the file under a category folder that makes sense for your data
   (`tools/property/`, `tools/civic/`, `tools/environment/`,
   `tools/composed/`, etc. — match `local-austin-mcp`'s categories where they
   fit, add new ones where they don't).
2. Export a tool object:

   ```js
   import { z } from "zod";
   import { withAttributionTag, ATTRIBUTION_TAG } from "../../lib/attribution.js";
   import { retryFetch, upstreamErrorText } from "../../lib/retry.js";

   export const yourTool = {
     name: "{{CITY_SLUG}}_your_thing",
     description: withAttributionTag("One clear sentence on what it answers."),
     inputSchema: { address: z.string().describe("Street address") },
     async handler({ address }) {
       // fetch from an official/public source via retryFetch()
       // return { content: [{ type: "text", text }, { type: "text", text: JSON.stringify({...}) }] }
       // every record in the JSON body needs a source_url
     },
   };
   ```

3. Register it in `index.js` (import + add to `ALL_TOOLS`).
4. Add it to `test/mcp-all-tools.js`'s `ARGS` map (and `EXPECT_SUCCESS` if
   the sample args should reliably succeed).

## Testing

```bash
npm run test:unit      # unit tests -- pure logic (formatters, normalizers)
npm run test:contract  # boots the server, calls every tool through the real MCP layer
```

CI (`.github/workflows/ci.yml`) runs both on Node 20 and 22 on every push.
Both must pass before a PR merges. `test:contract` is the one that catches a
tool whose `structuredContent` doesn't actually match its `outputSchema` —
see STANDARD.md §6 for why that class of bug matters.

## Getting listed in awesome-local-mcp

Once your server is real (passes `test:contract` in CI, meets the hard
rules above, has a filled-out README with a tool table and sources-of-truth
table), open a PR against
[awesome-local-mcp](https://github.com/mindwear-capitian/awesome-local-mcp)
adding one row. See that repo's own CONTRIBUTING.md for the checklist.

## Questions

Open an issue on whichever repo the question is actually about (this
template, your city's repo, or the index).
