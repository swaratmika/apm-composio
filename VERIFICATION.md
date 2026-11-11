# Verification log

Date of checks: 2026-11-06. This records what was checked for the five library guides, what changed as a result, and what is still unverified.

## How the checking was done

- Each guide was written only after reading the provider's official pages (GitHub Docs, Slack docs, Google Cloud Help, Microsoft Learn for LinkedIn, Shopify.dev). Third-party pages (Nango, WorkOS) were read as secondary sources and labeled that way.
- **Caveat on method:** pages were read through a fetch tool that returns a model-generated extract of the page for a given question, not the raw page. Numbers and wording in the guides come from those extracts. Before relying on any figure (especially review durations and thresholds), open the cited page and confirm it. This is the single biggest limit on the evidence quality here.
- `npm run check` applies one rule set to the library and to the live agent: a claim labeled "official docs" must cite an official source, "third-party" must cite a source, and "probe" requires a probe result. It also unit-tests the server-side pass that downgrades unsupported labels.

## What was corrected or changed

| # | Finding | Action |
|---|---|---|
| 1 | WorkOS (Aug 27, 2026) says Slack Marketplace submission needs 5+ active installs. Slack's review guide says 10+ active workspaces (28 days). | Guide follows Slack. The conflict is shown in "Conflicts and corrections" and the reader is told to confirm in their app's submission page. |
| 2 | Nango quotes Google as saying restricted-scope reviews take 4-6 weeks. Google's two verification pages that were read state no timelines. | Every Google timing is labeled third-party. The guide does not present it as a Google commitment. |
| 3 | Google test-mode limit (100 users) and assessor prices appear only in Nango's guide (prices dated Aug 2025). | Labeled third-party and flagged as possibly stale. |
| 4 | My first draft of the Slack guide listed a Slack URL as a source that was seen in search results but never opened. | Removed and citations reindexed. Only pages that were actually read are cited. |
| 5 | `npm run check` found three guides whose timeline carried an evidence label with no citation. | Added citation support to the timeline field and filled in sources. |
| 6 | Library guides cannot carry probe evidence. | Probe results live in `data/probes/` and are merged at load time, so they cannot be hand-labeled. |

## What could not be verified

- **Probes were not run.** The authoring sandbox blocks outbound traffic to the providers (HTTP 403 from the network proxy, or refused connections). Every library guide shows "Tested: not run". Run `npm run probe` on a normal network to record real results; they replace the placeholder automatically.
- **Live generation against the real model was not run.** There was no API key in the authoring environment. The agent loop, tool dispatch, validation, SSRF guards, SSE streaming and storage were run against a scripted mock of the Messages API (`scripts/mock-anthropic.mjs`). Output quality from the real model is untested.
- GitHub: whether organizations require approval of third-party OAuth apps (general knowledge, labeled uncertain). Client-secret steps were not on the fetched page.
- LinkedIn: approval timelines for gated products, and whether a Company Page association is needed to create an app. The official page read does not say.
- Shopify: App Store review duration and scope-related rejection reasons (not covered on the pages read). Forum thread titles hinting at "app is under review" install blocks for unlisted apps were seen but not read.
- Slack: rate-limit differences between unlisted and Marketplace apps (not on the pages read).
- Hands-on time figures ("10-15 minutes", "a few hours") are our own estimates and are labeled as such.

## Suggested spot-checks before submitting

1. Open Slack's Marketplace review guide and confirm: 10 active workspaces, 28 days, 10 business days preliminary, 10 weeks functional (6 weeks for updates).
2. Open Google's verification requirements page and confirm it states no review timelines and that the security assessment is annual.
3. Open Shopify's distribution page and confirm the "cannot change distribution method later" statement.
4. Run `npm run probe` from your own network and look at what each provider returns for a dummy client ID.
5. Generate two or three apps you have not seen live and read the "uncertain" and "relabeled" lines critically.
