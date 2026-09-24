---
starter_id: 10x-astro-starter
package_manager: npm
project_name: tribal-wars
hints:
  language_family: js
  team_size: solo
  deployment_target: render
  ci_provider: github-actions
  ci_default_flow: auto-deploy-on-merge
  bootstrapper_confidence: first-class
  path_taken: standard
  quality_override: false
  self_check_answers: null
  has_auth: true
  has_payments: false
  has_realtime: false
  has_ai: false
  has_background_jobs: true
---

## Why this stack

TribalWars is a small web-app with a 3-week after-hours first flow, login, and scheduled combat (queues + time windows). For web-app + JavaScript the registry default is 10x-astro-starter: typed Astro/React stack with auth and database included, which matches FR-001 and a solo shipping pace. You took the standard path with GitHub Actions CI; deployment was later moved from Vercel to an always-on Render web service (Node standalone adapter, auto-deploy on push to `master`), see infrastructure.md. Scaffolding confidence is first-class. Background jobs (battle windows, build/train queues) will run as a poller inside the always-on web process, with due items stored as Supabase rows so they survive deploys and restarts.
