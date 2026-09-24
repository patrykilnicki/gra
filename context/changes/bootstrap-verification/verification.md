---
bootstrapped_at: 2026-09-24T18:53:15Z
starter_id: 10x-astro-starter
starter_name: 10x Astro Starter (Astro + Supabase + Cloudflare)
project_name: tribal-wars
language_family: js
package_manager: npm
cwd_strategy: git-clone
bootstrapper_confidence: first-class
phase_3_status: ok
audit_command: npm audit --json
---

## Hand-off

```yaml
starter_id: 10x-astro-starter
package_manager: npm
project_name: tribal-wars
hints:
  language_family: js
  team_size: solo
  deployment_target: vercel
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
```

## Why this stack

TribalWars is a small web-app with a 3-week after-hours first flow, login, and scheduled combat (queues + time windows). For web-app + JavaScript the registry default is 10x-astro-starter: typed Astro/React stack with auth and database included, which matches FR-001 and a solo shipping pace. You took the standard path and set deploy to Vercel with GitHub Actions auto-deploy on merge. Scaffolding confidence is first-class; edge-oriented defaults mean battle windows and build/train queues may later need an explicit worker or scheduled job rather than long in-request work.

## Pre-scaffold verification

| Signal             | Value                                                       | Severity | Notes                                              |
| ------------------ | ----------------------------------------------------------- | -------- | -------------------------------------------------- |
| npm package        | not run                                                     | —        | cmd_template starts with git clone; npm view skipped |
| GitHub repo        | przeprogramowani/10x-astro-starter last pushed 2026-09-12   | fresh    | from card.docs_url                                 |

## Scaffold log

**Resolved invocation**: `git clone https://github.com/przeprogramowani/10x-astro-starter .bootstrap-scaffold && cd .bootstrap-scaffold && npm install`
**Strategy**: git-clone
**Exit code**: 0
**Files moved**: 30664
**Conflicts (.scaffold siblings)**: none
**.gitignore handling**: moved silently
**.bootstrap-scaffold cleanup**: deleted

Upstream `.git/` was deleted before files were moved up. `context/` in the current directory was left unchanged (the starter did not ship a `context/` directory).

npm install completed with `EBADENGINE` warnings: several packages require Node `^20.19.0 || ^22.13.0 || >=24` (and some `^22.22.3 || ^24.16.0 || >=26.3.0`). The local runtime was Node v22.12.0.

## Post-scaffold audit

**Tool**: npm audit --json
**Status**: failed to run
**Reason**: The configured npm registry `https://registry.npmmirror.com` returned 404 `[NOT_IMPLEMENTED] /-/npm/v1/security/* not implemented yet` for `POST /-/npm/v1/security/audits/quick`. Exit code 1. No vulnerability counts were produced.
**Partial output (if any)**:

```
{
  "message": "404 Not Found - POST https://registry.npmmirror.com/-/npm/v1/security/audits/quick - [NOT_IMPLEMENTED] /-/npm/v1/security/* not implemented yet",
  "statusCode": 404,
  "body": {
    "error": "[NOT_IMPLEMENTED] /-/npm/v1/security/* not implemented yet"
  }
}
```

## Hints recorded but not acted on

| Hint                       | Value                  |
| -------------------------- | ---------------------- |
| bootstrapper_confidence    | first-class            |
| quality_override           | false                  |
| path_taken                 | standard               |
| self_check_answers         | null                   |
| team_size                  | solo                   |
| deployment_target          | vercel                 |
| ci_provider                | github-actions         |
| ci_default_flow            | auto-deploy-on-merge   |
| has_auth                   | true                   |
| has_payments               | false                  |
| has_realtime               | false                  |
| has_ai                     | false                  |
| has_background_jobs        | true                   |

## Next steps

Next: a future skill will set up agent context (CLAUDE.md, AGENTS.md). For now, your project is scaffolded and verified — happy hacking.

Useful manual steps in the meantime:
- `git init` (if you have not already) to start your own repo history.
- Review any `.scaffold` siblings the conflict policy created and decide which version of each file to keep.
- Address audit findings per your project's risk tolerance — the full breakdown is in this log.
