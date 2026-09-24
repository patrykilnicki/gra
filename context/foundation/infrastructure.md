---
project: TribalWars
researched_at: 2026-09-24
recommended_platform: Render
runner_up: Railway
context_type: mvp
tech_stack:
  language: JavaScript
  framework: Astro 7.3 SSR + React 19
  runtime: Node.js 22.14.0
---

## Recommendation

**Deploy on Render.**

TribalWars needs a process that stays up between requests so battle windows and the build/train queue are not tied to a single HTTP call. Render’s paid web service is that process: one Node server in Frankfurt, Supabase left as the external database. The first-ranked fit was Railway; after the risk review, Render was selected instead. Developer experience outweighed minimum cost, users are in one region, and co-located databases were not required. Vercel, Netlify, and Cloudflare Workers were dropped because they do not keep a process alive between requests. The repo today uses `@astrojs/cloudflare` and Wrangler; production on Render means switching the adapter to `@astrojs/node` and leaving `npm run dev` as `astro dev`.

## Platform Comparison

Checked 2026-09-24. Pass / Partial / Fail against CLI-first ops, managed runtime, agent-readable docs, a stable deploy API, and MCP or CI integration.

| Platform | CLI-first | Managed/Serverless | Agent-readable docs | Stable deploy API | MCP / Integration | Persistent process | Shortlist |
|---|---|---|---|---|---|---|---|
| Cloudflare Workers | Pass | Pass | Pass | Pass | Pass | No (Cron, Queues, Durable Object alarms) | Dropped |
| Vercel | Pass | Pass | Pass | Partial | Partial (MCP public beta; rollback CLI public beta) | No (Fluid pauses; Hobby cron ≤1/day) | Dropped |
| Netlify | Partial | Pass | Pass | Partial | Pass | No (scheduled 30s, background 15 min) | Dropped |
| Railway | Partial | Pass | Pass | Partial | Pass (MCP GA) | Yes, unless Serverless sleep is on | Runner-up |
| Render | Partial | Pass | Pass | Partial | Pass | Yes on a paid web service; Free sleeps after 15 min | Recommended |
| Fly.io | Pass | Pass | Pass | Partial | Fail | Yes (Machines) | Third |

Cloudflare is the strongest match for the current scaffold (`@astrojs/cloudflare` 14, Wrangler 4, `wrangler deploy` / `wrangler rollback` / `wrangler tail`, docs at `llms.txt`). It cannot keep a Node process resident. Scheduled work is another invocation (Cron Triggers are GA; Free CPU is 10 ms per invocation). That fails the always-on constraint.

Vercel runs Astro SSR through `@astrojs/vercel` on Fluid compute (Node 22 supported). Instances pause when idle. Cron is GA but Hobby is limited to about once a day. The official MCP and CLI rollback were public beta on the check date. First-party Postgres and KV are sunset in favor of marketplace Neon and Upstash. Familiar, and the stack notes name Vercel, but it does not stay alive between requests.

Netlify runs Astro SSR on Functions via `@astrojs/netlify`, with `llms.txt` and a production MCP. There is no resident worker. Scheduled Functions cap at 30 seconds. Rollback is a dashboard publish or `netlify api restoreSiteDeploy`, not a single deploy-style command.

Railway would have been the default: always-on Node, `railway up` / `railway logs`, docs at `https://docs.railway.com/llms.txt`, MCP GA, Hobby about $5/month including $5 of usage, `eu-west` (Amsterdam) for a stateless service. Rollback of an arbitrary deploy is dashboard-only. Cron cannot be the web server, cannot run more often than every 5 minutes, and skips a tick if the previous run has not exited. Its Astro guide still shows Astro 4. Those risks are why it stayed the runner-up.

Fly.io Machines are a real always-on VM (`fly deploy`, `fly logs`, image rollback, `https://docs.fly.io/llms.txt`). There is no ongoing free allowance (trial only). A small always-on machine is on the order of $6/month before rootfs and IPv4 extras. Unmanaged Fly Postgres is unsupported. There is no deploy MCP (Sprites MCP is a different product). Docker and `fly.toml` are a rougher loop than Render for someone who already knows Vercel.

### Shortlisted Platforms

#### 1. Render (Recommended)

A paid web service in Frankfurt is a long-lived Node process with HTTP, WebSockets, and an optional separate cron job. Docs are markdown (`https://render.com/docs/llms.txt`). The CLI can create a service and tail logs (`render services create`, `render logs -r SERVICE_ID --tail`). An official MCP exists at `https://mcp.render.com/mcp`. Supabase stays outside Render. The gap versus Railway is that Git and the dashboard are still the primary path, and rollback is not a one-shot CLI command.

#### 2. Railway

Best day-to-day loop of the three that can stay up: CLI deploy, GA MCP, EU region, lower Hobby floor. It lost the final call on the risk review (dashboard-only rollback, 5-minute cron that must exit, stale Astro 4 guide, Serverless sleep if that toggle is left on).

#### 3. Fly.io

The most direct “process that never exits” model, and the CLI is the product. It scored third because the loop is a container and a TOML file, billing has several small meters (rootfs on stopped machines, volumes, dedicated IPv4), and there is no first-class deploy MCP. Multi-region is unnecessary for this MVP.

## Anti-Bias Cross-Check: Render

### Devil's Advocate — Weaknesses

1. Render’s Astro guide still offers a Free instance. Free web services spin down after about 15 minutes idle and take on the order of a minute to wake. A battle window or queue that assumes the process is there will miss players whenever the service is asleep. Staying up means a paid plan (Starter is about $7/month), not the free path in that guide.
2. Production moves from `@astrojs/cloudflare` and workerd to `@astrojs/node` standalone. `astro:env` secrets, Supabase cookie sessions, and `npm run smoke` are exercised today against the Cloudflare dev server. A green local smoke run does not prove the Node server on Render.
3. Services created from 21 April 2026 default to Node 24. This repo pins 22.14.0 in `.nvmrc`. A dashboard `NODE_VERSION` overrides `.nvmrc`. Shipping on 24 while developing on 22 is a silent runtime split.
4. Rollback is the dashboard or the API, and a dashboard rollback turns autodeploy off. There is no `render rollback` equivalent in the researched CLI. Supabase migrations do not revert when the web service does.
5. A Render cron job is a second service (about $1/month minimum, one run at a time, 12-hour cap). It is not the public web process. An in-memory `setInterval` inside the web service dies on every deploy and restart unless due battles are rows in Supabase that a poller can resume.

### Pre-Mortem — How This Could Fail

The service went up on Render’s Free instance because that is what the Astro page suggests, and the first evening of play looked fine. Overnight the process slept. The next attack was accepted by a cold start, then the in-memory queue was empty after the spin-down, so the report never appeared. Upgrading to Starter fixed the sleep and exposed the next gap: every git push restarted Node and dropped timers that had not been written to Supabase. A separate cron job was added, pointed at UTC, and the evening window in Poland landed late. The same week the dashboard Node default (24) overrode `.nvmrc` on one service only, and a React island build failed in Frankfurt while `astro dev` on Node 22 still passed. The bad deploy stayed live because rollback is a dashboard action that also disables autodeploy, and nobody noticed until the next morning. The $7 plan was never the expensive part. Treating Free as always-on, and memory as the battle clock, was.

### Unknown Unknowns

- Render’s Astro doc still mentions `output: 'hybrid'`, which Astro 5 removed. This app is already `output: "server"`. Do not reintroduce `hybrid`.
- `@astrojs/node` standalone reads `HOST` and `PORT`. Render injects `PORT` (default `10000`) and requires `HOST=0.0.0.0`. Binding `127.0.0.1` makes the service look healthy in logs and return errors at the proxy.
- `npm run dev` (`astro dev`) stays the local loop. It does not boot the Render runtime. The production command is `node dist/server/entry.mjs` after `astro build`. `astro preview` is not that server.
- `.nvmrc` is `22.14.0`. Render only honors it when `NODE_VERSION` is unset. Setting Node 24 in the dashboard wins.
- Persistent disks disable zero-downtime deploys and are unnecessary while Supabase holds game state. Do not add a disk to keep battle timers.
- Preview environments and the Cursor Origin integration were called out separately from GA Git deploys; Origin was beta on the check date. Do not assume a PR URL is a faithful always-on copy of production.

## Operational Story

- **Preview deploys**: Connect the GitHub repo and turn on autodeploy for the production branch. Pull-request previews are Render Preview Environments, a separate feature from the production web service. Do not run battle-timing checks on a Free preview; it sleeps. Fork PRs should not receive production secrets.
- **Secrets**: `SUPABASE_URL` and `SUPABASE_KEY` live as Render environment variables on the web service (available at build and runtime). They stay out of git (`.env` / `.dev.vars` remain local). Anyone with workspace access to the service can read them. Rotation is: update the variable, redeploy, then revoke the old Supabase key.
- **Rollback**: In the Dashboard, open the service → Events or Deploys → roll back to a previous successful deploy. That action disables autodeploy until it is turned back on. There is no dedicated CLI rollback. Expect about a minute, not an instant edge revert. Supabase SQL already applied does not roll back with the service; ship backward-compatible migrations.
- **Approval**: A human should create the paid service, attach the production custom domain, change instance type, disable autodeploy, and rotate Supabase keys. An agent may tail logs and trigger a deploy hook or `render deploys create` only after the service id and an API key already exist.
- **Logs**: `render logs -r SERVICE_ID --tail` for the running service (Render CLI, logged in). Build logs are on the deploy event in the Dashboard and via the same CLI. The docs MCP at `https://mcp.render.com/mcp` is for platform docs, not a substitute for those runtime logs.

## Risk Register

| Risk | Source | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| Free instance sleeps after ~15 min and drops in-memory queues | Devil's advocate | High | High | Create the web service on a paid plan (Starter). Never use Free for the battle resolver. |
| In-process timers vanish on deploy or restart | Pre-mortem | High | High | Store due battles and queue items in Supabase. On boot, poll rows whose window has elapsed. |
| Node 24 dashboard default overrides `.nvmrc` 22.14.0 | Unknown unknowns | Medium | Medium | Do not set `NODE_VERSION`. Confirm the service resolved 22.14.0 from `.nvmrc` before the first real deploy. |
| `@astrojs/cloudflare` behavior differs from `@astrojs/node` (cookies, `astro:env`, smoke) | Devil's advocate | Medium | High | Switch the adapter in a branch, run `astro build` and `HOST=0.0.0.0 node dist/server/entry.mjs` locally, then re-run the auth smoke against that server. |
| App rollback does not undo Supabase migrations; CLI has no one-shot rollback | Devil's advocate | Medium | High | Keep migrations additive. Practice one dashboard rollback on a staging service and re-enable autodeploy afterward. |
| Cron UTC drift or overlap skips a battle window | Pre-mortem | Medium | Medium | Prefer a poller inside the always-on web process over a separate cron. If cron is used, schedule in UTC and make the job exit. |
| Official Astro-on-Render steps suggest Free and mention removed `hybrid` output | Unknown unknowns | Medium | Medium | Follow this file’s Getting Started, not the Free-instance snippet in the platform guide. |
| Frankfurt service with a distant Supabase region | Research finding | Medium | Medium | Place the Render service in Frankfurt and the Supabase project in an EU region. |

## Getting Started

This repo is Astro 7.3.2 (`output: "server"`), `@astrojs/cloudflare` 14, Wrangler 4, and Node 22.14.0 (`.nvmrc`). Render’s generic Astro page is written around an older output mode and a Free instance. Use the steps below.

1. Keep local development on `npm run dev` (`astro dev`). Do not add a Render-specific dev server.
2. Switch the production adapter off Cloudflare and onto Node standalone: `npx astro add node`, then set `output: "server"` and `adapter: node({ mode: "standalone" })`. Remove the Cloudflare adapter from `astro.config.mjs`. Add a start script that does not replace `dev`: `"start": "node ./dist/server/entry.mjs"`.
3. Prove the Node server locally: `npm run build`, then `HOST=0.0.0.0 PORT=10000 npm start`. Hit a logged-in route. `astro preview` does not substitute for this.
4. Create a **Web Service** (not a static site) in **Frankfurt**, on a **paid** instance (Starter), build `npm install && npm run build`, start `node ./dist/server/entry.mjs`. Set `HOST=0.0.0.0`. Leave `NODE_VERSION` unset so `.nvmrc` selects 22.14.0. Set `SUPABASE_URL` and `SUPABASE_KEY` on the service.
5. Confirm in the deploy log that Node resolved to 22.14.0, then run `npm run smoke` with `BASE_URL` set to the Render URL.

## Out of Scope

The following were not evaluated in this research:
- Docker image configuration
- CI/CD pipeline setup
- Production-scale architecture (multi-region, HA, DR)
