# OAuth Path

Look up an app and get a guide you can follow and share for registering its OAuth app and getting it approved: how hard it is, how long it takes, what can block you, and what an agent can do versus where you take over. Submit any app name or website and an agent researches it live and generates a new guide.

## What's in the box

- **Guide library** (5 hand-checked guides): GitHub, Slack, Google (Gmail), LinkedIn, Shopify. Each is a JSON file in `data/guides/`.
- **Live generation**: `POST /api/generate` runs a Claude agent that searches the web, reads official docs, optionally probes the provider's authorize endpoint with a dummy client ID, and submits a structured guide. Progress streams to the page; the result gets a shareable URL (`/g/<slug>`).
- **Share formats**: link, Copy as Markdown, Download JSON, Print / save PDF.
- **Verification log**: [VERIFICATION.md](./VERIFICATION.md) lists what was checked, corrected, and left unverified. Read it first.

## Key decisions

1. **Difficulty depends on what you need, so every guide has "paths".** Slack unlisted distribution is easy; Slack Marketplace is a multi-week review with a 10-active-workspace prerequisite. Google non-sensitive scopes need no verification; Gmail read scopes need verification plus an annual security assessment. One number per app would hide this, so each guide gives an overall score and a score per path.
2. **Hands-on effort and waiting are separate.** Developers plan differently for "two days of work" and "ten weeks in someone's queue". Review durations are shown as the provider's stated upper bounds and are never presented as typical.
3. **Every statement carries an evidence label**: *official docs* (read on an official page), *probe* (observed by a test request), *third-party* (only a non-official source says so), *uncertain* (estimate, inference, or unknown). Unknown is shown as unknown, not as a guess.
4. **The code enforces the labels, not the model.** On live guides, the server downgrades "official docs" claims that cite no official source, downgrades "probe" claims when no probe succeeded, drops invalid citations, and builds the "what was tested" section from real tool results. Downgrades are disclosed on the page. `npm run check` applies the same rules to the library.
5. **Steps are split by owner**: *agent can do*, *you do* (needs your identity, domain, account, money or legal responsibility), *provider / waiting*. This answers "what can an agent do for me and where do I take over".
6. **Focused scope over breadth.** Five apps chosen to span the kinds of gate: none (GitHub), optional vs required review (Slack), restricted scopes with a paid annual audit (Google), partner-gated products (LinkedIn), and a permanent distribution choice (Shopify). The schema, the generator and the integrity checks are the scalable part; more guides are produced by running the generator and having a human review them.
7. **Probes are deliberately weak.** An unauthenticated request with a dummy client ID can show that an endpoint exists and how an unknown client is rejected. It cannot show approval time or whether an app would be approved. Probe results are stored as "informational", never "passed".

## Run it

```bash
npm install
cp .env.example .env.local     # set ANTHROPIC_API_KEY for live generation
npm run dev                    # http://localhost:3000
```

| Command | What it does |
|---|---|
| `npm run check` | Lints library guides against the evidence rules and unit-tests the integrity pass |
| `npm run probe` | Sends unauthenticated probes to the five providers' authorize endpoints and records results in `data/probes/` (needs open outbound HTTPS). They appear in each guide's "tested" section |
| `node scripts/mock-anthropic.mjs 4010` | Scripted fake Messages API. Run the app with `ANTHROPIC_BASE_URL=http://localhost:4010 ANTHROPIC_API_KEY=test` to exercise the agent loop without a model |

No credentials are in the repo. `.env*` is git-ignored.

### Deploy

Any Node host works (tested locally with `next build && next start`). On Vercel, set `ANTHROPIC_API_KEY`; the generate route sets `maxDuration = 300`, so use a plan that allows long functions. Optional: `ANTHROPIC_MODEL` (default `claude-sonnet-5-5`), `GENERATE_LIMIT_PER_HOUR` (default 6 per IP).

## How live generation works

`lib/agent.ts` runs a tool-use loop with four tools: the server-side `web_search`, `fetch_page` (SSRF-guarded: https only, public addresses only, bounded size, time and redirects), `probe_oauth` (max 3, same guards), and `submit_guide` (JSON-schema validated; an incomplete guide is rejected with the reason so the model can fix it). Budgets: 8 searches, 10 page fetches, 3 probes, 16 turns. The system prompt tells the agent to treat page content as untrusted data.

## Limitations

- **The real model was not run during development.** The agent loop, tools, validation, SSE streaming and storage were exercised against a mock API. Quality of live output on unfamiliar apps is untested; try several before demoing and read the "uncertain" and "relabeled" lines.
- **No probe results ship with the library.** The authoring sandbox blocked outbound traffic to providers, so each library guide says "Tested: not run" until you run `npm run probe`.
- **No browser-driven setup attempt.** The agent does not sign up or click through provider consoles. "Tested" means unauthenticated endpoint probes only. A real setup attempt (creating a dev app, hitting a review form) is the most valuable evidence to add next.
- **Docs were read through an extraction tool**, not raw pages. Spot-check figures in VERIFICATION.md before relying on them.
- **Live guides are not durable on serverless hosts.** They are written to a temp directory (or `GUIDE_STORE_DIR`). If the server copy is gone, the page falls back to the copy in the generating browser's local storage and says so. Swap `lib/store.ts` for KV or blob storage for permanent links.
- The rate limiter is in-memory and per instance. Hands-on hour figures are estimates. Guides are a snapshot dated on the page; provider policies change (the Slack threshold discrepancy found here is an example).
- Not legal or compliance advice.

## Next steps if this were a product

- Show "Composio manages this OAuth app for you" when a toolkit has managed OAuth, so the guide only appears when you actually need your own app (Composio's managed-auth docs describe a `composio_managed_auth_schemes` field for this check).
- Re-verify guides on a schedule and show the age of each claim.
- A review queue for generated guides before they enter the library, and anonymous "this was wrong / this worked for me" reports with dates, since real approval times are the biggest gap.
- Browser-agent setup attempts in a sandbox, with screenshots as evidence.
