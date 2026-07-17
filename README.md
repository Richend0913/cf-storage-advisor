# Cloudflare Storage Advisor (AI-Powered)

**Live tool:** https://cf-storage-advisor.burningbros.workers.dev

A free tool that answers: *"Should I use Workers KV, D1, R2, or Durable Objects for this?"*

Describe what you're building and what data you need to store, optionally add constraints (read/write ratio, consistency needs, data size), and an AI model recommends the best-fitting Cloudflare storage product — with reasoning tied to the specific facts (consistency model, latency, limits) that make it fit.

## Why this exists

Cloudflare's own storage-options docs and various blog posts already have static comparison tables — the same table for every reader. Nothing found matches your specific use case description to a personalized, reasoned recommendation. This tool is grounded (RAG-style) in a curated table of Cloudflare's own documented facts about Workers KV, D1, R2, and Durable Objects, and the model is explicitly instructed to only use those facts — not invent limits, pricing, or behavior that aren't listed, and not recommend any Cloudflare product outside this list.

## How it works

- A hardcoded table of the 4 core Cloudflare storage products, each with consistency model, latency characteristics, ideal use cases, key limits, and a link to the source doc.
- User describes their use case in plain language (+ optional constraints).
- [Cloudflare Workers AI](https://developers.cloudflare.com/workers-ai/) (`@cf/meta/llama-3.1-8b-instruct-fp8-fast`) is called with a system prompt restricting it to the documented facts, asked to recommend one primary product (and optionally one alternative if genuinely ambiguous), or say what extra detail it would need instead of guessing.
- A small daily quota (tracked in Workers KV) caps total AI calls per day so the tool stays inside Cloudflare Workers AI's free Neuron allowance even under heavy or abusive traffic.

## Stack

- Single [Cloudflare Worker](https://developers.cloudflare.com/workers/) (`worker.js`), no framework, no build step.
- Bindings: Workers AI (`env.AI`) + one Workers KV namespace for the daily quota counter.
- Deploy with [Wrangler](https://developers.cloudflare.com/workers/wrangler/):

```bash
npx wrangler kv namespace create QUOTA   # then put the returned id in wrangler.toml
npx wrangler deploy
```

## Keeping the storage facts current

Cloudflare occasionally changes limits, pricing, or adds new storage products. To refresh:

1. Check https://developers.cloudflare.com/workers/platform/storage-options/ and the individual product docs (KV/D1/R2/Durable Objects).
2. Update the `STORAGE_DB` array in `worker.js` (keep entries sourced only from official docs).
3. Redeploy.

PRs that add more official-doc-sourced facts or fix bugs are welcome.

## Traffic

The Cloudflare GraphQL Analytics API isn't reachable from this project's deploy token (no `Account Analytics:Read` scope), so the Worker counts its own aggregate page views in KV: see `/stats` for the live numbers. It's a same-origin request counter only — no cookies, no per-visitor identifiers. Requests sending an `X-Skip-Analytics: 1` header or a common bot/test User-Agent (curl, Playwright, etc.) aren't counted. `/stats` also publishes up to 20 deduped raw `User-Agent` strings per day (no IP, no cookies) for hits that passed the bot filter, so an unexplained non-zero day can be checked against known bot signatures instead of assumed to be human traffic.

## License

MIT — see [LICENSE](LICENSE).

## Related tools

Other free Cloudflare tools from the same project:

- [Workers AI Free Tier Neuron Calculator](https://workers-ai-cost-calculator.burningbros.workers.dev/) ([source](https://github.com/Richend0913/workers-ai-cost-calculator))
- [Cloudflare Error Code AI Explainer](https://cf-error-explainer.burningbros.workers.dev/) ([source](https://github.com/Richend0913/cf-error-explainer))
- [Cloudflare Async Processing Advisor (Queues vs Workflows vs Durable Objects vs Cron)](https://cf-async-advisor.burningbros.workers.dev/) ([source](https://github.com/Richend0913/cf-async-advisor))
- [Cloudflare Page Rule Migration Advisor](https://cf-pagerule-migrator.burningbros.workers.dev/) ([source](https://github.com/Richend0913/cf-pagerule-migrator))

---

Built by an AI-run micro-tool project ([BURNING AUTONOMY](https://github.com/Richend0913)). Independent, unofficial — not affiliated with or endorsed by Cloudflare, Inc. No signup, no per-visitor tracking — aggregate page-view counts only, published live at `/stats`.
